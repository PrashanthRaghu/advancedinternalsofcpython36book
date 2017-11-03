# Advanced  Compilation Concepts
The global state of the compilation is held by the following structure.

  - Listing .1 The compiler structure
>  struct compiler { <br>
>PyObject *c_filename;<br>
>struct symtable *c_st;<br>
>PyFutureFeatures *c_future; /* pointer to module's __future__ */<br>
>PyCompilerFlags *c_flags;<br>
>int c_optimize; /* optimization level */<br>
>int c_interactive; /* true if in interactive mode */<br>
>int c_nestlevel;<br>
>struct compiler_unit *u; /* compiler state for current block */<br>
>PyObject *c_stack; /* Python list holding compiler_unit ptrs<br>
>*/<br>
>PyArena *c_arena; /* pointer to memory allocation arena */<br>
>};<br>

The compilation state for the current block of code is held by the structure compiler_unit
>struct compiler_unit {<br>
>PySTEntryObject *u_ste;<br>
>PyObject *u_name;<br>
>PyObject *u_qualname; /* dot-separated qualified name (lazy) */<br>
>int u_scope_type;<br>
>/* The following fields are dicts that map objects to<br>
>the index of them in co_XXX. The index is used as<br>
>the argument for opcodes that refer to those collections.<br>
>*/<br>
>PyObject *u_consts; /* all constants */<br>
>PyObject *u_names; /* all names */<br>
>PyObject *u_varnames; /* local variables */<br>
>PyObject *u_cellvars; /* cell variables */<br>
>PyObject *u_freevars; /* free variables */<br>
>PyObject *u_private; /* for private name mangling */<br>
>Py_ssize_t u_argcount; /* number of arguments for block */<br>
>Py_ssize_t u_kwonlyargcount; /* number of keyword only arguments for<br>
>block */<br>
>/* Pointer to the most recently allocated block. By following b_list<br>
>members, you can reach all early allocated blocks. */<br>
>basicblock *u_blocks;<br>
>basicblock *u_curblock; /* pointer to current block */<br>
>int u_nfblocks;<br>
>struct fblockinfo u_fblock[CO_MAXBLOCKS];<br>
>int u_firstlineno; /* the first lineno of the block */<br>
>int u_lineno; /* the lineno for the current stmt */<br>
>int u_col_offset; /* the offset of the current stmt */<br>
>int u_lineno_set; /* boolean to indicate whether instr<br>
>has been generated with current lineno */<br>
>};<br>

Very similar to the symbol table entries the compiler units are stacked in the compiler data
structure containing the nested blocks.
The generated opcode is stored within the structure basicblock:

>typedef struct basicblock_ {<br>
>/* Each basicblock in a compilation unit is linked via b_list in the <br>
>reverse order that the block are allocated. b_list points to the<br>
>next block, not to be confused with b_next, which is next by control<br>
>flow. */<br>
>struct basicblock_ *b_list;<br>
>/* number of instructions used */<br>
>int b_iused;<br>
>/* length of instruction array (b_instr) */<br>
>int b_ialloc;<br>
>/* pointer to an array of instructions, initially NULL */<br>
>struct instr *b_instr;<br>
>/* If b_next is non-NULL, it is a pointer to the next<br>
>block reached by normal control flow. */<br>
>struct basicblock_ *b_next;<br>
>/* b_seen is used to perform a DFS of basicblocks. */<br>
>unsigned b_seen : 1;<br>
>/* b_return is true if a RETURN_VALUE opcode is inserted. */<br>
>unsigned b_return : 1;<br>
>/* depth of stack upon entry of block, computed by stackdepth() */<br>
>int b_startdepth;<br>
>/* instruction offset for block, computed by assemble_jump_offsets() */<br>
>int b_offset;<br>
>} basicblock;<br>

The basic block contains instructions for every scope in the program.
Now let us examine which units of code have their own compiler_units
For the creation of a new compiler unit the following function is called
compile.c line no 531

>static int<br>
compiler_enter_scope(struct compiler *c, identifier name,<br>
int scope_type, void *key, int lineno)<br>
{<br>
struct compiler_unit *u;<br>
basicblock *block;<br>
u = (struct compiler_unit *)PyObject_Malloc(sizeof(<br>
struct compiler_unit));<br>
if (!u) {<br>
PyErr_NoMemory();<br>
return 0;<br>
}<br>
memset(u, 0, sizeof(struct compiler_unit));<br>
u->u_scope_type = scope_type;<br>
u->u_argcount = 0;<br>
u->u_kwonlyargcount = 0;<br>
u->u_ste = PySymtable_Lookup(c->c_st, key);<br>
if (!u->u_ste) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
Py_INCREF(name);<br>
u->u_name = name;<br>
u->u_varnames = list2dict(u->u_ste->ste_varnames);<br>
u->u_cellvars = dictbytype(u->u_ste->ste_symbols, CELL, 0, 0);<br>
if (!u->u_varnames || !u->u_cellvars) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
if (u->u_ste->ste_needs_class_closure) {<br>
/* Cook up an implicit __class__ cell. */<br>
_Py_IDENTIFIER(__class__);<br>
PyObject *tuple, *name, *zero;<br>
int res;<br>
assert(u->u_scope_type == COMPILER_SCOPE_CLASS);<br>
assert(PyDict_Size(u->u_cellvars) == 0);<br>
name = _PyUnicode_FromId(&PyId___class__);<br>
if (!name) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
tuple = _PyCode_ConstantKey(name);<br>
if (!tuple) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
zero = PyLong_FromLong(0);<br>
if (!zero) {<br>
Py_DECREF(tuple);<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
res = PyDict_SetItem(u->u_cellvars, tuple, zero);<br>
Py_DECREF(tuple);<br>
Py_DECREF(zero);<br>
if (res < 0) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
}<br>
u->u_freevars = dictbytype(u->u_ste->ste_symbols, FREE, DEF_FREE_CLASS,<br>
PyDict_Size(u->u_cellvars));<br>
if (!u->u_freevars) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
u->u_blocks = NULL;<br>
u->u_nfblocks = 0;<br>
u->u_firstlineno = lineno;<br>
u->u_lineno = 0;<br>
u->u_col_offset = 0;<br>
u->u_lineno_set = 0;<br>
u->u_consts = PyDict_New();<br>
if (!u->u_consts) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
u->u_names = PyDict_New();<br>
if (!u->u_names) {<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
u->u_private = NULL;<br>
/* Push the old compiler<br>_unit on the stack. */<br>
if (c->u) {<br>
PyObject *capsule = PyCapsule_New(c->u, CAPSULE_NAME, NULL);<br>
if (!capsule || PyList_Append(c->c_stack, capsule) < 0) {<br>
Py_XDECREF(capsule);<br>
compiler_unit_free(u);<br>
return 0;<br>
}<br>
Py_DECREF(capsule);<br>
u->u_private = c->u->u_private;<br>
Py_XINCREF(u->u_private);<br>
}<br>
c->u = u;<br>
c->c_nestlevel++;<br>
block = compiler_new_block(c);<br>
if (block == NULL)<br>
return 0;<br>
c->u->u_curblock = block;<br>
if (u->u_scope_type != COMPILER_SCOPE_MODULE) {<br>
if (!compiler_set_qualname(c))<br>
return 0;<br>
}<br>
return 1;<br>
}<br>
if (!compiler_enter_scope(c, module, COMPILER_SCOPE_MODULE, mod, 0))<br>
return NULL;<br>
Purpose: For the module<br>
if (!compiler_enter_scope(c, name, scope_type, (void *)s, s->lineno)) {<br>
return 0;<br>
}<br>
Purpose: For a function
if (!compiler_enter_scope(c, s->v.ClassDef.name,<br>
COMPILER_SCOPE_CLASS, (void *)s, s->lineno))<br>
return 0;<br>
Purpose: For a class<br>
if (!compiler_enter_scope(c, name, COMPILER_SCOPE_LAMBDA,<br>
(void *)e, e->lineno))<br>
Purpose: For a lambda<br>
if (!compiler_enter_scope(c, name, COMPILER_SCOPE_COMPREHENSION,<br>
(void *)e, e->lineno))<br>
{<br>
goto error;<br>
}<br>
>Purpose: For a comprehension<br>

Very similar to symbol tables we see that a module , class , function, lambda and
comprehension have their own compiler units.
Let us understand the creation of basic blocks.
Compile.c line no 754

>static basicblock *<br>
compiler_new_block(struct compiler *c)<br>
{<br>
basicblock *b;<br>
struct compiler_unit *u;<br>
u = c->u;<br>
b = (basicblock *)PyObject_Malloc(sizeof(basicblock));<br>
if (b == NULL) {<br>
PyErr_NoMemory();<br>
return NULL;<br>
}<br>
memset((void *)b, 0, sizeof(basicblock));<br>
/* Extend the singly linked list of blocks with new block. */<br>
b->b_list = u->u_blocks;<br>
u->u_blocks = b;<br>
return b;<br>
>}<br>

The basicblocks of code in the compiler unit form a reverse linked list with the entry block at
the end of the list.
That’s about the workings of the compilers seems very less but it will be complemented by
the next two chapters.

