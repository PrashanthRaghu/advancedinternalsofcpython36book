<h1 align="center" > chapter 9 </h1>
<h2 align="center" > Exception Handling </h2>
 
Let us understand exception handling through a very basic example i.e when an operation like 
division throws an error. 

```python
>>> a = 100 / 0
Traceback(most recent call last):
 File "<stdin>", line 1, in <module>
ZeroDivisionError : division by zero
```
 
Let us trace this error path. 
 
Ceval.c line no 1385 
```C 
TARGET(BINARY_TRUE_DIVIDE) {
 	PyObject *divisor = POP();
 	PyObject *dividend = TOP();
 	PyObject *quotient = PyNumber_TrueDivide(dividend, divisor);
	Py_DECREF(dividend);
 	Py_DECREF(divisor);
 	SET_TOP(quotient);
 	if(quotient == NULL)
 	   goto error;
 	DISPATCH();
 }
error:

	 assert(why == WHY_NOT);
 why = WHY_EXCEPTION;

 /* Double-check exception status. */
#ifdef NDEBUG
 	if(! PyErr_Occurred())
 	PyErr_SetString(PyExc_SystemError,
 			"error return without exception set");
#else
 	assert(PyErr_Occurred());
#endif

 	/* Log traceback info. */
 	PyTraceBack_Here(f); 
	if(tstate->c_tracefunc != NULL)
 		call_exc_trace(tstate->c_tracefunc, tstate->c_traceobj,
 		tstate, f);

exit_eval_frame:
    if(PyDTrace_FUNCTION_RETURN_ENABLED())
 	dtrace_function_return(f);
    Py_LeaveRecursiveCall();
    f->f_executing = 0;
    tstate->frame = f->f_back;
```
 
We see that in the simplest case of execution the interpreter loop is terminated and since we 
are in the interactive mode of python the command prompt waits for the next command from 
the user. 
 
<p style="color:blue"> Exercise: </p>
 
Create a program with multiple threads and raise an exception in one of these threads and 
check if all the other threads are suspended or they continue execution. 
 
**Topic 9.1 Exception handling with exception handlers** 
 
Let us take a simple example 
 
```C
try:
    a = 100 /0
except:
    print(" We know it ' s an exception ")
 
  1	 	        0 SETUP_EXCEPT		     12 (to 14)
 
  2	        	2 LOAD_CONST		      0 (100)
                4 LOAD_CONST              1 (0)
                6 BINARY_TRUE_DIVIDE
                8 STORE_NAME              0 (a)
	           10 POP_BLOCK
               12 JUMP_FORWARD           20(to 34)
  3       >>   14 POP_TOP
 	           16 POP_TOP
 	           18 POP_TOP
  4            20 LOAD_NAME               1(print)
               22 LOAD_CONST              2('We know it's an exception')
               24 CALL_FUNCTION           1 
               26 POP_TOP
               28 POP_EXCEPT
               30 JUMP_FORWARD            2(to 34)
               32 END_FINALLY
          >>   34 LOAD_CONST              3(None)
               36 RETURN_VALUE
while(why != WHY_NOT && f->f_iblock > 0) {
	 /* Peek at the current block. */
	 PyTryBlock *b = & f->f_blockstack[f->f_iblock - 1]; 

	 assert(why != WHY_YIELD);
	if(b->b_type == SETUP_LOOP && why == WHY_CONTINUE) {
		 why = WHY_NOT;
		 JUMPTO(PyLong_AS_LONG(retval));
		 Py_DECREF(retval);
 	  	 break;
	}
	/* Now we have to pop the block. */
	f->f_iblock--;

	if(b->b_type == EXCEPT_HANDLER) {
 		UNWIND_EXCEPT_HANDLER(b);
 		continue;
	}
	UNWIND_BLOCK(b);
	if(b->b_type == SETUP_LOOP && why == WHY_BREAK) {
		why = WHY_NOT;
 		JUMPTO(b->b_handler);
 		break;
	}
	if(why == WHY_EXCEPTION &&(b->b_type == SETUP_EXCEPT
 	   || b->b_type == SETUP_FINALLY)) {
 		PyObject *exc, *val, *tb;
 	   int handler = b->b_handler;
	   /* Beware, this invalidates all b->b_* fields */
	   PyFrame_BlockSetup(f, EXCEPT_HANDLER, - 1, STACK_LEVEL());
	   PUSH(tstate->exc_traceback);
	   PUSH(tstate->exc_value);
	   if(tstate->exc_type != NULL) {
		 PUSH(tstate->exc_type);
	   }
	   else {
		 Py_INCREF(Py_None);
		 PUSH(Py_None);
	   }
	   PyErr_Fetch(&exc, &val, &tb);

 	/* Make the raw exception data
	 available to the handler,
	 so a program can emulate the
	 Python main loop . */
	 PyErr_NormalizeException(
		 & exc, & val, & tb);
	 if(tb != NULL)
		 PyException_SetTraceback(val, tb);
	 else
		 PyException_SetTraceback(val, Py_None);
	 Py_INCREF(exc);
	 tstate->exc_type = exc;
	 Py_INCREF(val);
	 tstate->exc_value = val;
	 tstate->exc_traceback = tb;
	 if(tb == NULL)
		 tb = Py_None;
	 Py_INCREF(tb);
	 PUSH(tb);
	 PUSH(val);
	 PUSH(exc);
	 why = WHY_NOT;
	 JUMPTO(handler);
	 break;
	}
```
	 
Let us try to understand this piece of code flow through opcodes and code flow 
 
Observation 1  
```C 
SETUP_EXCEPT		 12(to 14)

TARGET(SETUP_LOOP)
	 TARGET(SETUP_EXCEPT)
	 TARGET(SETUP_FINALLY) {
		 /* NOTE: If you add any new block-setup opcodes that
		 are not try /except/ finally handlers, you may need
		 to update the PyGen_NeedsFinalizing() function.
		 */

		 PyFrame_BlockSetup(f, opcode, INSTR_OFFSET() + oparg,
 			STACK_LEVEL());
	 DISPATCH();
 }
```

Create a new jump block with the handler set to the jump offset 14, which is the code flow for 
the exception. 
 
Observation 2  

```C 
12 JUMP_FORWARD 20(to 34)
```
 
If no exception occurs skip the exception code flow path. 
 
Observation 3  
 
```C
 	 PyObject *exc, *val, *tb;
	 int handler = b->b_handler;
	 /* Beware, this invalidates all b->b_* fields */
	 PyFrame_BlockSetup(f, EXCEPT_HANDLER, - 1, STACK_LEVEL());
	 PUSH(tstate->exc_traceback);
	 PUSH(tstate->exc_value);
	 if(tstate->exc_type != NULL) {
		 PUSH(tstate->exc_type);
 	 }
	 else {
		 Py_INCREF(Py_None);
		 PUSH(Py_None);
	 }
	 PyErr_Fetch(& exc, & val, & tb);
	 /* Make the raw exception data
	    available to the handler,
	    so a program can emulate the
	    Python main loop . */
	 PyErr_NormalizeException(
 		&exc, &val, &tb);
	 if(tb != NULL)
		 PyException_SetTraceback(val, tb);
	 else
		 PyException_SetTraceback(val, Py_None);
	 Py_INCREF(exc);
	 tstate->exc_type = exc;
	 Py_INCREF(val);
	 tstate->exc_value = val;
	 tstate->exc_traceback = tb;
	 if(tb == NULL)
		 tb = Py_None;
	 Py_INCREF(tb);
	 PUSH(tb);
	 PUSH(val);
	 PUSH(exc); 
	 why = WHY_NOT;
 	 JUMPTO(handler);
	 break;

void
PyErr_Fetch(PyObject ** p_type, PyObject ** p_value, PyObject ** p_traceback)
{
   PyThreadState *tstate = PyThreadState_GET();

   *p_type = tstate->curexc_type;
   *p_value = tstate->curexc_value;
   *p_traceback = tstate->curexc_traceback;
   tstate->curexc_type = NULL;
   tstate->curexc_value = NULL;
   tstate->curexc_traceback = NULL;
}
```
 
Explanation 
 
Fetch the current exception from the thread state and push it onto the stack and replace the 
old exception on the current thread state by the new exception. Most importantly jump the 
handler. 
 
Observation 4 

```C 
 3 	>>	14 POP_TOP
		16 POP_TOP
		18 POP_TOP
		20 LOAD_NAME	 1(print)
		22 LOAD_CONST	 2('We know it's an exception')
		CALL_FUNCTION    1
```
 
Pop the current exception off the stack and perform the exception handler. 
 
**Topic 9.2 How is the exception set into the current thread state** 
 
The function where the error occurred raises the exception and set’s it onto the current 
thread by calling the function below. 

```C 
void PyErr_Restore(PyObject *type, PyObject *value, PyObject *traceback)
  {
	 PyThreadState *tstate = PyThreadState_GET();
	 PyObject *oldtype, *oldvalue, *oldtraceback;
	 if(traceback != NULL && ! PyTraceBack_Check(traceback)) {
	 /* XXX Should never happen -- fatal error instead? */
	 /* Well, it could be None. */
		 Py_DECREF(traceback);
		 traceback = NULL;
  }

 /* Save these in locals to safeguard against recursive
    invocation through Py_XDECREF */
 oldtype = tstate->curexc_type;
 oldvalue = tstate->curexc_value;
 oldtraceback = tstate->curexc_traceback;

 tstate->curexc_type = type;
 tstate->curexc_value = value;
 tstate->curexc_traceback = traceback;

 Py_XDECREF(oldtype);
 Py_XDECREF(oldvalue);
 Py_XDECREF(oldtraceback);
}
```

Let us turn back to our example 
 
```C 
if(b_size == 0) {
 	PyErr_SetString(PyExc_ZeroDivisionError,
 			"division by zero");
	 goto error;
 }
 // longobject.c line no 3750

void
PyErr_SetString(PyObject *exception, const char *string)
  {
 	 PyObject *value = PyUnicode_FromString(string);
	 PyErr_SetObject(exception, value);
	 Py_XDECREF(value);
  }

void PyErr_SetObject(PyObject *exception, PyObject *value)
  {
 	PyThreadState *tstate = PyThreadState_GET();
 	PyObject *exc_value;
 	PyObject *tb = NULL;
  if (exception != NULL &&
	 ! PyExceptionClass_Check(exception)) {
	 PyErr_Format(PyExc_SystemError,
		 "exception %R not a BaseException subclass",
		 exception);
	 return;
 } 

 Py_XINCREF(value);
 exc_value = tstate->exc_value;
 if(exc_value != NULL && exc_value != Py_None) {
	 /* Implicit exception chaining */
	 Py_INCREF(exc_value);
	 if(value == NULL || ! PyExceptionInstance_Check(value)) {
		 /* We must normalize the value right now */
		 PyObject *fixed_value;

		 /* Issue #23571: functions must not be called with an
		 exception set */
		 PyErr_Clear();

		fixed_value = _PyErr_CreateException(exception, value);
		Py_XDECREF(value);
		if(fixed_value == NULL) {
		    return;
		}

	 value = fixed_value;
 }


 /* Avoid reference cycles through the context chain.
    This is O(chain length) but context chains are
    usually very short . Sensitive readers may try
    to inline the call to PyException_GetContext . */
 if(exc_value != value) {
    PyObject *o = exc_value, *context;
    while((context = PyException_GetContext(o))) {
 	Py_DECREF(context);
	 if(context == value) {
	     PyException_SetContext(o, NULL);
	     break;
	 }
	 o = context;
     }
     PyException_SetContext(value, exc_value);
  }
 else {
	Py_DECREF(exc_value);
     }
 }
 if(value != NULL && PyExceptionInstance_Check(value))
 	tb = PyException_GetTraceback(value);
 Py_XINCREF(exception);
 PyErr_Restore(exception, value, tb);
}
```
 
We see that the function that raises the exception sets the exception on the current thread. 
 
**Topic 9.2 Raise an exception**
 
Let us consider the simple example 

```C 
try:
	 raise ValueError("This is the exception")
except:
	 print("Exception")

   1		 0 SETUP_EXCEPT		    12 (to 14)
   2		 2 LOAD_NAME		     0 (ValueError)
	     	 4 LOAD_CONST		     0 ('This is the exception')
  		     6 CALL_FUNCTION		 1
             8 RAISE_VARARGS		 1
	        10 POP_BLOCK
 	        12 JUMP_FORWARD		    20 (to 34) 
   
   3    >>  14 POP_TOP
	        16 POP_TOP
	        18 POP_TOP

   4 	>>	20 LOAD_NAME		     1 (print)
		    22 LOAD_CONST		     1 ('Exception')
		    24 CALL_FUNCTION	     1
		    26 POP_TOP
		    28 POP_EXCEPT
 		    30 JUMP_FORWARD	  	     2 (to 34)
		    32 END_FINALLY
	 >>     34 LOAD_CONST	 	     2 (None)
		    36 RETURN_VALUE
```
 
Let us examine the opcode implementation for the RAISE_VARARGS 

```C 
 TARGET(RAISE_VARARGS) {
	 PyObject *cause = NULL, *exc = NULL;
	 switch(oparg) {
	 case 2:
		 cause = POP(); /* cause */
	 case 1:
		 exc = POP(); /* exc */
	 case 0 : /* Fallthrough */
		 if(do_raise(exc, cause)) {
			 why = WHY_EXCEPTION;
			 goto fast_block_end;
		 }
		 break;
	 default:
		 PyErr_SetString(PyExc_SystemError,
		 "bad RAISE_VARARGS oparg");
		 break;
	 } 
	 goto error;
   }

static int
do_raise(PyObject *exc, PyObject *cause)
 {
 	
	PyObject *type = NULL, *value = NULL;
     if (exc == NULL) {
	 /* Reraise */
	 PyThreadState *tstate = PyThreadState_GET();
	 PyObject *tb;
	 type = tstate->exc_type;
	 value = tstate->exc_value;
	 tb = tstate->exc_traceback;
	 if(type == Py_None || type == NULL) {
	     PyErr_SetString(PyExc_RuntimeError,
			 "No active exception to reraise");
	      return 0;
         }
	 Py_XINCREF(type);
	 Py_XINCREF(value);
	 Py_XINCREF(tb);
	 PyErr_Restore(type, value, tb);
	 return 1;
  }

 /* We support the following forms of raise:
 raise raise <instance>
 raise <type> */

 if(PyExceptionClass_Check(exc)) {
     type = exc;
     value = PyObject_CallObject(exc, NULL);
     if(value == NULL)
	 goto raise_error;
     if(! PyExceptionInstance_Check(value)) {
	 PyErr_Format(PyExc_TypeError,
		 "calling %R should have returned an instance of "
		 "BaseException, not %R",
		 type, Py_TYPE(value));
	 goto raise_error;
     }
 }
 else if(PyExceptionInstance_Check(exc)) {
	 value = exc;
	 type = PyExceptionInstance_Class(exc);
	 Py_INCREF(type);
 }
 else {
	   /* Not something you can raise. You get an exception
	    anyway, just not what you specified :-) */
      	 Py_DECREF(exc);
	 PyErr_SetString(PyExc_TypeError,
		 "exceptions must derive from BaseException");
	 goto raise_error;
      } 

 if (cause) {
     PyObject *fixed_cause;
     if(PyExceptionClass_Check(cause)) {
	 fixed_cause = PyObject_CallObject(cause, NULL);
	 if(fixed_cause == NULL)
		 goto raise_error;
	 Py_DECREF(cause);
     }
     else if(PyExceptionInstance_Check(cause)) {
	 fixed_cause = cause;
     }
     else if(cause == Py_None) {
	 Py_DECREF(cause);
	 fixed_cause = NULL;
     }
 else {
	 PyErr_SetString(PyExc_TypeError,
			"exception causes must derive from "
			 "BaseException");
	 goto raise_error;
 }
     PyException_SetCause(value, fixed_cause);
}

 PyErr_SetObject(type, value); // 1
 /* PyErr_SetObject incref's its arguments */
 Py_XDECREF(value);
 Py_XDECREF(type);
 return 0;

raise_error:
   Py_XDECREF(value);
   Py_XDECREF(type);
   Py_XDECREF(cause);
   return 0;
}
```
Observation 1  
 
Raise the exception on the current thread state and go to the normal exception handling code 
flow. 
 
<p style="color:blue"> Exercise </p>
 
Debug this code flow and examine the cases for different types of errors. 
