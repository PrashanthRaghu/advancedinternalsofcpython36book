<h1 align="center"> Chapter  7  </h1>
 
<h2 align="center"> Assembler  </h2>

### The assembler in python is called to compile code units into their separate piece of code object.

_Listing 6.1 The PyCodeObject_
```python

typedef struct {
PyObject_HEAD
int co_argcount ; /* #arguments, except *args */
int co_kwonlyargcount ; /* #keyword only arguments */
int co_nlocals ; /* #local variables */
int co_stacksize ; /* #entries needed for evaluation stack */
int co_flags ; /* CO_..., see below */
int co_firstlineno ; /* first source line number */
PyObject * co_code ; /* instruction opcodes */
PyObject * co_consts ; /* list (constants used) */
PyObject * co_names ; /* list of strings (names used) */
PyObject * co_varnames ; /* tuple of strings (local variable names) */
PyObject * co_freevars ; /* tuple of strings (free variable names) */
PyObject * co_cellvars ; /* tuple of strings (cell variable names)
*/
/* The rest aren't used in either hash or comparisons, except for
co_name,
used in both . This is done to preserve the name and line number
for tracebacks and debuggers ; otherwise , constant de - duplication
would collapse identical functions / lambdas defined on different
lines.
*/
unsigned char * co_cell2arg ; /* Maps cell vars which are arguments. */
PyObject * co_filename ; /* unicode (where it was loaded from) */
PyObject * co_name ; /* unicode (name, for reference) */
PyObject * co_lnotab ; /* string (encoding addr<->lineno mapping)
See
Objects / lnotab_notes . txt for details . */
void * co_zombieframe ; /* for optimization only (see frameobject.c)
*/
PyObject * co_weakreflist ; /* to support weakrefs to code objects */
/* Scratch space for extra data relating to the code object.
Type is a void * to keep the format private in codeobject . c to force
people to go through the proper APIs . */
void * co_extra;
} PyCodeObject;
```

The assembler combines the basicblocks of a compiler_unit into a code string named co_code.

_Listing 6.2 Places where the assembler is called._
```python

1. co = assemble ( c , addNone ); // compile.c line no 1510 in function
compiler_mod
2. co = assemble ( c , 1 ); // compile.c line no 1886 in function
compiler_function
3. co = assemble ( c , 1 ); // compile.c line no 2003 in function
compiler_class
4. co = assemble ( c , 0 ); // compile.c line no 2093 in function
compiler_lambda
5. co = assemble ( c , 1 ); // compile.c line no 3929 in function
compiler_comprehension
```
Now we understand that along with the module a function, class, lambda and comprehensions have their own code objects which are compiled by the assembler from the compiler_unit of the block of code.

Let us try to understand the important functions of the assembler.

_Listing 6.3 The assembler function_

**Compile.c line no 5315**
```python

static PyCodeObject *
assemble ( struct compiler * c , int addNone)
{
basicblock * b , * entryblock;
struct assembler a;
int i , j , nblocks;
PyCodeObject * co = NULL;
/* Make sure every block that falls off the end returns None.
XXX NEXT_BLOCK () isn 't quite right, because if the last
block ends with a jump or return b_next shouldn 't set.
*/
if (! c -> u -> u_curblock -> b_return ) {
NEXT_BLOCK ( c );
if ( addNone)
ADDOP_O ( c , LOAD_CONST , Py_None , consts );
ADDOP ( c , RETURN_VALUE );
}
nblocks = 0;
entryblock = NULL ;
for ( b = c -> u -> u_blocks ; b != NULL ; b = b -> b_list ) {
nblocks ++;
entryblock = b ; // 1
}
/* Set firstlineno if it wasn't explicitly set. */
if (! c -> u -> u_firstlineno ) {
if ( entryblock && entryblock -> b_instr &&
entryblock -> b_instr -> i_lineno)
c -> u -> u_firstlineno = entryblock -> b_instr -> i_lineno;
else
c -> u -> u_firstlineno = 1;
}
if (! assemble_init (& a , nblocks , c -> u -> u_firstlineno )) // 2
goto error;
dfs ( c , entryblock , & a ); // 3
/* Can't modify the bytecode after computing jump offsets. */
assemble_jump_offsets (& a , c ); // 4
/* Emit code in reverse postorder from dfs. */
for ( i = a . a_nblocks - 1 ; i >= 0 ; i --) {
b = a . a_postorder [ i ];
for ( j = 0 ; j < b -> b_iused ; j ++)
if (! assemble_emit (& a , & b -> b_instr [ j ])) // 5
goto error;
}
if ( _PyBytes_Resize (& a . a_lnotab , a . a_lnotab_off ) < 0)
goto error;
if ( _PyBytes_Resize (& a . a_bytecode , a . a_offset * sizeof ( _Py_CODEUNIT )) <
0)
goto error;
co = makecode ( c , & a ); // 6
error:
assemble_free (& a );
return co;
}
```

Observations from _Listing 6.3_

1. Previously we have understood that the code blocks are arranged as a reverse linked list with the first block at the end of the linked list. Hence it is traversed and the entry block is computed along with the length of the list of blocks.

2. 
    ```python 3.6.3
		static int
		assemble_init ( struct assembler * a , int nblocks , int firstlineno)
		{
		memset ( a , 0 , sizeof ( struct assembler )); // 1
		a -> a_lineno = firstlineno;
		a -> a_bytecode = PyBytes_FromStringAndSize ( NULL ,
		DEFAULT_CODE_SIZE ); // 2
		if (! a -> a_bytecode)
		return 0;
		a -> a_lnotab = PyBytes_FromStringAndSize ( NULL ,
		DEFAULT_LNOTAB_SIZE );
		if (! a -> a_lnotab)
		return 0;
		if (( size_t ) nblocks > SIZE_MAX / sizeof ( basicblock *)) {
		PyErr_NoMemory ();
		return 0;
		}
		a -> a_postorder = ( basicblock **) PyObject_Malloc(
		sizeof ( basicblock *) *
		nblocks ); //3
		if (! a -> a_postorder ) {
		PyErr_NoMemory ();
		return 0;
		}
		return 1;
		}
		
		1. Initialize all bytes of the assembler to NULL

		2. Initialize the bytecode to NULL

		3. Allocate memory to the postorder basicblocks pointer array.
	  ```
3.  
    ```python		
		static void
		dfs ( struct compiler * c , basicblock * b , struct assembler * a)
		{
		int i;
		struct instr * instr = NULL;
		if ( b -> b_seen)
		return;			
		b -> b_seen = 1 ; // 1
		if ( b -> b_next != NULL)
		dfs ( c , b -> b_next , a ); // 2
		for ( i = 0 ; i < b -> b_iused ; i ++) {
		instr = & b -> b_instr [ i ];
		if ( instr -> i_jrel || instr -> i_jabs)
		dfs ( c , instr -> i_target , a ); // 3
		}
		a -> a_postorder [ a -> a_nblocks ++] = b ; // 4
		}
	
		1. b_seen is the recursion breaker

		2. Performs a depth first recursion on the code blocks of the compiler_unit

		3. dfs is also performed on the jump targets as we have seen from the previous chapter.
	
		4. The blocks are added into the post order array.
    ```

4.  
    ```python
            static void
		assemble_jump_offsets ( struct assembler * a , struct compiler * c)
		{
		basicblock * b;
		int bsize , totsize , extended_arg_recompile;
		int i;
		/* Compute the size of each block and fixup jump args. Replace block pointer with position in bytecode . */
		do {
		totsize = 0;
		for ( i = a -> a_nblocks - 1 ; i >= 0 ; i --) {
		b = a -> a_postorder [ i ];
		bsize = blocksize ( b );
		b -> b_offset = totsize;
		totsize += bsize;
		}
		extended_arg_recompile = 0;
		for ( b = c -> u -> u_blocks ; b != NULL ; b = b -> b_list ) {
		bsize = b -> b_offset;
		for ( i = 0 ; i < b -> b_iused ; i ++) {
		struct instr * instr = & b -> b_instr [ i ];
		int isize = instrsize ( instr -> i_oparg );
		/* Relative jumps are computed relative to the instruction pointer after fetching the jump instruction. */
		bsize += isize;
		if ( instr -> i_jabs || instr -> i_jrel ) {
		instr -> i_oparg = instr -> i_target -> b_offset;
		if ( instr -> i_jrel ) {
		instr -> i_oparg -= bsize;
		}
		instr -> i_oparg *= sizeof ( _Py_CODEUNIT ); // 1
		if ( instrsize ( instr -> i_oparg ) != isize ) {
		extended_arg_recompile = 1;
		}
		}
		}
		}
		
		/* XXX: This is an awful hack that could hurt performance, but on the bright side it should work until we come up with a better solution. The issue is that in the first loop blocksize () is called which calls instrsize () which requires i_oparg be set appropriately . There is a bootstrap problem because i_oparg is calculated in the second loop above. So we loop until we stop seeing new EXTENDED_ARGs. The only EXTENDED_ARGs that could be popping up are ones in jump instructions . So this should converge fairly quickly. */
		
		} while ( extended_arg_recompile );
		}
		
		1. Add the jump offset as the oparg
		
		TARGET ( JUMP_FORWARD ) {
		JUMPBY ( oparg );
		FAST_DISPATCH ();
		} 						// ceval.c 2882
		
		#define JUMPBY ( x ) ( next_instr += ( x ) /
		sizeof ( _Py_CODEUNIT ))
	  ```
5.  
    ```python
		static int
		assemble_emit ( struct assembler * a , struct instr * i)
		{
		int size , arg = 0;
		Py_ssize_t len = PyBytes_GET_SIZE ( a -> a_bytecode );
		_Py_CODEUNIT * code;
		arg = i -> i_oparg;
		size = instrsize ( arg );
		if ( i -> i_lineno && ! assemble_lnotab ( a , i )) // 1
		return 0;
		if ( a -> a_offset + size >= len / ( int ) sizeof ( _Py_CODEUNIT )) {
		if ( len > PY_SSIZE_T_MAX / 2)
		return 0;
		if ( _PyBytes_Resize (& a -> a_bytecode , len * 2 ) < 0)
		return 0;
		}
		code = ( _Py_CODEUNIT *) PyBytes_AS_STRING ( a -> a_bytecode ) +
		a -> a_offset ; // 2
		a -> a_offset += size;
		write_op_arg ( code , i -> i_opcode , arg , size ); // 3
		return 1 ;
		}

		1. Add the line no offset to the assembler

		2. Compute the current offset to write the instruction

		3. Write the instruction as a string
    ```

6.  
    ```python
		static PyCodeObject *
		makecode ( struct compiler * c , struct assembler * a)
		{	
		PyObject * tmp;
		PyCodeObject * co = NULL;
		PyObject * consts = NULL;
		PyObject * names = NULL;
		PyObject * varnames = NULL;
		PyObject * name = NULL;
		PyObject * freevars = NULL;
		PyObject * cellvars = NULL;
		PyObject * bytecode = NULL;
		Py_ssize_t nlocals;
		int nlocals_int;
		int flags;
		int argcount , kwonlyargcount;
		tmp = dict_keys_inorder ( c -> u -> u_consts , 0 );
		if (! tmp)
		goto error;
		consts = PySequence_List ( tmp ); /* optimize_code requires a list */
		Py_DECREF ( tmp );
		names = dict_keys_inorder ( c -> u -> u_names , 0 );
		varnames = dict_keys_inorder ( c -> u -> u_varnames , 0 );
		if (! consts || ! names || ! varnames)
		goto error;
		cellvars = dict_keys_inorder ( c -> u -> u_cellvars , 0 );
		if (! cellvars)
		goto error;
		freevars = dict_keys_inorder ( c -> u -> u_freevars ,
		PyTuple_Size ( cellvars ));
		if (! freevars)
		goto error;
		nlocals = PyDict_Size ( c -> u -> u_varnames );
		assert ( nlocals < INT_MAX );
		nlocals_int = Py_SAFE_DOWNCAST ( nlocals , Py_ssize_t , int );
		flags = compute_code_flags ( c );
		if ( flags < 0)
		goto error;
		bytecode = PyCode_Optimize ( a -> a_bytecode , consts , names ,
		a -> a_lnotab ); // 1
		if (! bytecode)
		goto error;
		tmp = PyList_AsTuple ( consts ); /* PyCode_New requires a tuple */
		if (! tmp)
		goto error;
		Py_DECREF ( consts );
		consts = tmp;
		argcount = Py_SAFE_DOWNCAST ( c -> u -> u_argcount , Py_ssize_t , int );
		kwonlyargcount = Py_SAFE_DOWNCAST ( c -> u -> u_kwonlyargcount ,
		Py_ssize_t , int );
		co = PyCode_New ( argcount , kwonlyargcount,
		nlocals_int , stackdepth ( c ), flags,
		bytecode , consts , names , varnames,
		freevars , cellvars,
		c -> c_filename , c -> u -> u_name,
		c -> u -> u_firstlineno,
		a -> a_lnotab ); // 2
		error:
		Py_XDECREF ( consts );
		Py_XDECREF ( names );
		Py_XDECREF ( varnames );
		Py_XDECREF ( name );
		Py_XDECREF ( freevars );
		Py_XDECREF ( cellvars );
		Py_XDECREF ( bytecode );
		return co;
		}
		
		1. Optimize the code object.
		
		2. Construct the new code object for the block of code.
    ```

That’s all we have about assemblers in python. Hope you enjoyed it :).

