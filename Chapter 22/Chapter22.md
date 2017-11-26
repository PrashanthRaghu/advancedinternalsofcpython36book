<h1 align="center"> Chapter  22  </h1>
 
<h2 align="center"> Understanding Lambdas  </h2>

### Listing .1 Representation of the lambda object

```
y = lambda x : x * 2
Opcode
1 0 LOAD_CONST 0 (< code object <lambda> at 0x7f1e16b4e700 , file "test.py" , line 1 >)
2 LOAD_CONST 1 ( '<lambda>')
4 MAKE_FUNCTION 0
6 STORE_NAME 0 ( y)
8 LOAD_CONST 2 ( None)
10 RETURN_VALUE
```

_So we see that lambdas are functions created with the name ‘lambda’. As we had explained in the chapter on compilers we see that the lambda has it’s own code object._

_Lambda objects are stored in memory exactly like functions using the PyFunctionObject._

### Topic Compilation of a lambda

```
static int
compiler_lambda ( struct compiler * c , expr_ty e)
{
PyCodeObject * co;
PyObject * qualname;
static identifier name;
Py_ssize_t funcflags;
arguments_ty args = e -> v . Lambda . args;
assert ( e -> kind == Lambda_kind );
if (! name ) { 
name = PyUnicode_InternFromString ( "<lambda>" );
if (! name)
return 0;
}
funcflags = compiler_default_arguments ( c , args );
if ( funcflags == - 1 ) {
return 0;
}
if (! compiler_enter_scope ( c , name , COMPILER_SCOPE_LAMBDA, ( void *) e , e -> lineno )) // 1
return 0;    

/* Make None the first constant, so the lambda can't have a docstring . */

if ( compiler_add_o ( c , c -> u -> u_consts , Py_None ) < 0)
return 0;
c -> u -> u_argcount = asdl_seq_LEN ( args -> args ); 				// 2
c -> u -> u_kwonlyargcount = asdl_seq_LEN ( args -> kwonlyargs );
VISIT_IN_SCOPE ( c , expr , e -> v . Lambda . body ); 				// 3
if ( c -> u -> u_ste -> ste_generator ) {	
co = assemble ( c , 0 );
}
else {
ADDOP_IN_SCOPE ( c , RETURN_VALUE ); 			// 4
co = assemble ( c , 1 );
}
qualname = c -> u -> u_qualname;
Py_INCREF ( qualname );
compiler_exit_scope ( c );
if ( co == NULL)
return 0;
compiler_make_closure ( c , co , funcflags , qualname ); 			// 5
Py_DECREF ( qualname );
Py_DECREF ( co );
return 1;
}
```

**Observation 1**

_Create a new scope for the lambda function_

**Observation 2**

_Compute the number of arguments for the lambda_

**Observation 3**

_Compile the single expression of the lambda_

**Observation 4**

_Add a return value to the lambda and assemble the lambda_

**Observation 5**

_Make a closure to the parent scope_

### Topic Execution of a lambda

```
y = lambda x : x * 2
z = y ( 10)
1 0 LOAD_CONST 0 (< code object <lambda> at
0x7f8521377700 , file "test.py" , line 1 >)
2 LOAD_CONST 1 ( '<lambda>')
4 MAKE_FUNCTION 0
6 STORE_NAME 0 ( y)
3 8 LOAD_NAME 0 ( y)
10 LOAD_CONST 2 ( 10)
12 CALL_FUNCTION 1
14 STORE_NAME 1 ( z)
16 LOAD_CONST 3 ( None)
18 RETURN_VALUE
```

_The lambdas are called just like any other functions using the CALL_FUNCTION opcode. That’s it about lambdas it was just a very small chapter._ 

_Hope you liked it._
