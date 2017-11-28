<h1 align="center"> Chapter  25  </h1>
 
<h2 align="center"> Functioning of the import statement  </h2>

Let us create a file test1.py

```C
def func ():
		pass
```

Let us create another file test.py

```C
import test1
```

Let us get the opcode

```C
1 			0 LOAD_CONST 			0( 0)
			2 LOAD_CONST 			1( None)
			4 IMPORT_NAME 			0( test1)
			6 STORE_NAME 			0( test1)
```

Let us see the implementation of the opcode **IMPORT_NAME**

```C
TARGET ( IMPORT_NAME ) {
			PyObject * name = GETITEM ( names , oparg );
			PyObject * fromlist = POP ();
			PyObject * level = TOP ();
			PyObject * res;
			res = import_name ( f , name , fromlist , level );
			Py_DECREF ( level );
			Py_DECREF ( fromlist );
			SET_TOP ( res );
			if ( res == NULL)
			goto error;
			DISPATCH ();
		}

static PyObject *
import_name ( PyFrameObject * f , PyObject * name , PyObject * fromlist , PyObject
* level)
{
	_Py_IDENTIFIER ( __import__ );
	PyObject * import_func , * res;
	PyObject * stack [ 5 ];
	import_func = _PyDict_GetItemId ( f -> f_builtins , & PyId___import__ );
	if ( import_func == NULL ) {
	PyErr_SetString ( PyExc_ImportError , "__import__ not found" );
	return NULL;
	}

	/* Fast path for not overloaded __import__. */
	if ( import_func == PyThreadState_GET ()-> interp -> import_func ) {
		int ilevel = _PyLong_AsInt ( level );
		if ( ilevel == - 1 && PyErr_Occurred ()) {
		return NULL;
		}
		res = PyImport_ImportModuleLevelObject(name, f -> f_globals, f -> f_locals == NULL ? Py_None : f -> f_locals, fromlist,ilevel );
		return res;
	}

	Py_INCREF ( import_func );

	stack [ 0 ] = name;
	stack [ 1 ] = f -> f_globals;
	stack [ 2 ] = f -> f_locals == NULL ? Py_None : f -> f_locals;
	stack [ 3 ] = fromlist;
	stack [ 4 ] = level;
	res = _PyObject_FastCall ( import_func , stack , 5 );
	Py_DECREF ( import_func );
	return res;
}

PyObject *
PyImport_ImportModuleLevelObject ( PyObject * name , PyObject * globals , PyObject * locals , PyObject * fromlist , int level )
{
	_Py_IDENTIFIER ( _find_and_load );
	_Py_IDENTIFIER ( _handle_fromlist );
	PyObject * abs_name = NULL ;
	PyObject * final_mod = NULL ;
	PyObject * mod = NULL ;
	PyObject * package = NULL ;
	PyInterpreterState * interp = PyThreadState_GET ()-> interp ;
	int has_from ;
	
	if ( name == NULL ) {
		PyErr_SetString ( PyExc_ValueError , "Empty module name" );
		goto error ;
	}	

	/* The below code is importlib.__import__() & _gcd_import(), ported to C for added performance. */

	if (! PyUnicode_Check ( name )) {
		PyErr_SetString ( PyExc_TypeError , "module name must be a string" );
		goto error ;
	}
	if ( PyUnicode_READY ( name ) < 0 ) {
		goto error ;
	}
	if ( level < 0 ) {
		PyErr_SetString ( PyExc_ValueError , "level must be >= 0" );
		goto error ;
	}
	if ( level > 0 ) {
		abs_name = resolve_name ( name , globals , level );
		if ( abs_name == NULL )
			goto error ;
	}
	else { /* level == 0 */
		if ( PyUnicode_GET_LENGTH ( name ) == 0 ) {
			PyErr_SetString ( PyExc_ValueError , "Empty module name" );
			goto error ;
		}
		abs_name = name ;
		Py_INCREF ( abs_name );
	}

	mod = PyDict_GetItem ( interp -> modules , abs_name );
	if ( mod == Py_None ) {
		PyObject * msg = PyUnicode_FromFormat ( "import of %R halted; " "None in sys.modules" , abs_name );
		if ( msg != NULL ) {
			PyErr_SetImportErrorSubclass ( PyExc_ModuleNotFoundError , msg , abs_name , NULL ); Py_DECREF ( msg );
		}
		mod = NULL ;
		goto error ;
	}
	else if ( mod != NULL ) {
		_Py_IDENTIFIER ( __spec__ );
		_Py_IDENTIFIER ( _initializing );
		_Py_IDENTIFIER ( _lock_unlock_module );
		PyObject * value = NULL ;
		PyObject * spec ;
		int initializing = 0 ;

		Py_INCREF ( mod );
		/* Optimization: only call _bootstrap._lock_unlock_module() if
			__spec__._initializing is true.
			NOTE: because of this, initializing must be set *before*
			stuffing the new module in sys.modules.
		*/
		spec = _PyObject_GetAttrId ( mod , & PyId___spec__ );
		if ( spec != NULL ) {
			value = _PyObject_GetAttrId ( spec , & PyId__initializing );
			Py_DECREF ( spec );
		}
		if ( value == NULL )
			PyErr_Clear ();
		else {
			initializing = PyObject_IsTrue ( value );
			Py_DECREF ( value );
			if ( initializing == - 1 )
				PyErr_Clear ();
			if ( initializing > 0 ) {
#ifdef WITH_THREAD
				_PyImport_AcquireLock ();
#endif
			/* _bootstrap._lock_unlock_module() releases the import lock */

value = _PyObject_CallMethodIdObjArgs ( interp -> importlib , & PyId__lock_unlock_module , abs_name , NULL );

			if ( value == NULL )
				goto error ;
			Py_DECREF ( value );
		}
	}
}

	else {
#ifdef WITH_THREAD
		_PyImport_AcquireLock ();
#endif
				/* _bootstrap._find_and_load() releases the import lock */

		mod = _PyObject_CallMethodIdObjArgs ( interp -> importlib , & PyId__find_and_load , abs_name , interp -> import_func , NULL );
		if ( mod == NULL ) {
			goto error ;
		}
	}
	has_from = 0 ;
	if ( fromlist != NULL && fromlist != Py_None ) {
		has_from = PyObject_IsTrue ( fromlist );
		if ( has_from < 0 )
				goto error ;
	}
	if (! has_from ) {
		Py_ssize_t len = PyUnicode_GET_LENGTH ( name );
		if ( level == 0 || len > 0 ) {
			Py_ssize_t dot ;

			dot = PyUnicode_FindChar ( name , '.' , 0 , len , 1 );
			if ( dot == - 2 ) {
				goto error ;
			}
			if ( dot == - 1 ) {
									/* No dot in module name, simple exit */
				final_mod = mod ;
				Py_INCREF ( mod );
				goto error ;
			}
			if ( level == 0 ) {
				PyObject * front = PyUnicode_Substring ( name , 0 , dot );
				if ( front == NULL ) {
					goto error ;
				}	
				
				final_mod = PyImport_ImportModuleLevelObject ( front , NULL , NULL , NULL , 0 );
				Py_DECREF ( front );
			}
			else {
				  Py_ssize_t cut_off = len - dot ;
				  Py_ssize_t abs_name_len = PyUnicode_GET_LENGTH ( abs_name );
				  PyObject * to_return = PyUnicode_Substring ( abs_name , 0 , abs_name_len - cut_off );
				
				  if ( to_return == NULL ) {
						goto error ;
				  }
					final_mod = PyDict_GetItem ( interp -> modules , to_return );
					Py_DECREF ( to_return );
					if ( final_mod == NULL ) {
					PyErr_Format ( PyExc_KeyError , "%R not in sys.modules as expected" , to_return );
					goto error ;
				  }
				   Py_INCREF ( final_mod );
			}
		}
		else {
				final_mod = mod ;
				Py_INCREF ( mod );
		}
	}
	else {
			final_mod = _PyObject_CallMethodIdObjArgs ( interp -> importlib ,& PyId__handle_fromlist , mod , fromlist , interp -> import_func , NULL );
	}

error :
	Py_XDECREF ( abs_name );
	Py_XDECREF ( mod );
	Py_XDECREF ( package );
	if ( final_mod == NULL )
	remove_importlib_frames ();
	return final_mod ;
}
```

Let us consider the two important factors while importing a module.

**Observation 1**

```C
mod = PyDict_GetItem ( interp -> modules , abs_name );
```

The module is imported from the global module sets stored in the interpreter state.

**Observation 2**

```C
final_mod = _PyObject_CallMethodIdObjArgs ( interp -> importlib, & PyId__handle_fromlist , mod, fromlist , interp -> import_func, NULL );
```

The final module is extracted from the module by extracting the module using the importlib
function of the interpreter.

Let us understand what the importlib function is actually

pylifecycle.c line no 248

```C
importlib = PyImport_AddModule ( "_frozen_importlib" );
	if ( importlib == NULL ) {
		Py_FatalError ( "Py_Initialize: couldn't get _frozen_importlib from " "sys.modules" );
	}
	interp -> importlib = importlib;
	Py_INCREF ( interp -> importlib );

	interp -> import_func = PyDict_GetItemString ( interp -> builtins , "__import__" );
```

The importlib is actually the module _frozen_importlib.

Let us now understand what is a Module object in memory

```C
typedef struct {
	PyObject_HEAD
	PyObject * md_dict;
	struct PyModuleDef * md_def;
	void * md_state;
	PyObject * md_weaklist;
	PyObject * md_name ; 		/* for logging purposes after md_dict is cleared */
} PyModuleObject;
```

Let us now understand how built in modules are initialized.

```C
PyObject *
PyModule_Create2 ( struct PyModuleDef * module , int module_api_version)
{
	const char * name;
	PyModuleObject * m;
	PyInterpreterState * interp = PyThreadState_Get ()-> interp;
	if ( interp -> modules == NULL)
		Py_FatalError ( "Python import machinery not initialized" );
	if (! PyModuleDef_Init ( module ))
		return NULL;
	name = module -> m_name;
	if (! check_api_version ( name , module_api_version )) {
		return NULL;
	}
	if ( module -> m_slots ) {
		PyErr_Format( PyExc_SystemError, "module %s: PyModule_Create is incompatible with m_slots" , name );
		return NULL;
	}
	/* Make sure name is fully qualified.

		This is a bit of a hack : when the shared library is loaded,
		the module name is "package.module" , but the module calls
		PyModule_Create *() with just "module" for the name . The shared
		library loader squirrels away the true name of the module in
		_Py_PackageContext , and PyModule_Create *() will substitute this
		( if the name actually matches ).
	*/

	if ( _Py_PackageContext != NULL ) {
		char * p = strrchr ( _Py_PackageContext , '.' );
		if ( p != NULL && strcmp ( module -> m_name , p + 1 ) == 0 ) {
			name = _Py_PackageContext;
			_Py_PackageContext = NULL;
		}
	}
	if (( m = ( PyModuleObject *) PyModule_New ( name )) == NULL)
		return NULL;
	
	if ( module -> m_size > 0 ) {
		m -> md_state = PyMem_MALLOC ( module -> m_size );
		if (! m -> md_state ) {
			PyErr_NoMemory ();
			Py_DECREF ( m );
			return NULL;
		}
		memset ( m -> md_state , 0 , module -> m_size );
	}

	if ( module -> m_methods != NULL ) {
		if ( PyModule_AddFunctions (( PyObject *) m , module -> m_methods ) != 0 )
{
			Py_DECREF ( m );
			return NULL;
		}
	}
	if ( module -> m_doc != NULL ) {
		if ( PyModule_SetDocString (( PyObject *) m , module -> m_doc ) != 0 ) {
			Py_DECREF ( m );
			return NULL;
		}
	}	
	m -> md_def = module;
	return ( PyObject *) m;
}
```

Let us now understand the initialization of some builtin modules and the sys module.

```C
PyObject *
_PyBuiltin_Init ( void)
{
	PyObject * mod , * dict , * debug;

	if ( PyType_Ready (& PyFilter_Type ) < 0 ||
		PyType_Ready (& PyMap_Type ) < 0 ||
		PyType_Ready (& PyZip_Type ) < 0)
		return NULL;

	mod = PyModule_Create (& builtinsmodule );
	if ( mod == NULL)
		return NULL;
	dict = PyModule_GetDict ( mod );

#ifdef Py_TRACE_REFS
	/* "builtins" exposes a number of statically allocated objects
	 * that , before this code was added in 2.3 , never showed up in
	 * the list of "all objects" maintained by Py_TRACE_REFS . As a
	 * result , programs leaking references to None and False ( etc)
	 * couldn 't be diagnosed by examining sys.getobjects(0).
	 */
#define ADD_TO_ALL ( OBJECT ) _Py_AddToAllObjects (( PyObject *)( OBJECT ), 0)
#else
#define ADD_TO_ALL ( OBJECT ) ( void )0
#endif

#define SETBUILTIN ( NAME , OBJECT ) \
	if ( PyDict_SetItemString ( dict , NAME , ( PyObject *) OBJECT ) < 0 ) \
		return NULL ; \
	ADD_TO_ALL ( OBJECT)

	SETBUILTIN ( "None" , 				Py_None );
	SETBUILTIN ( "Ellipsis" , 			Py_Ellipsis );
	SETBUILTIN ( "NotImplemented" , 		Py_NotImplemented );
	SETBUILTIN ( "False" , 				Py_False );
	SETBUILTIN ( "True" , 				Py_True );
	SETBUILTIN ( "bool" , 				& PyBool_Type );
	SETBUILTIN ( "memoryview" , 			& PyMemoryView_Type );
	SETBUILTIN ( "bytearray" , 			& PyByteArray_Type );
	SETBUILTIN ( "bytes" , 				& PyBytes_Type );
	SETBUILTIN ( "classmethod" , 			& PyClassMethod_Type );
	SETBUILTIN ( "complex" , 			& PyComplex_Type );
	SETBUILTIN ( "dict" , 				& PyDict_Type );
	SETBUILTIN ( "enumerate" , 			& PyEnum_Type );
	SETBUILTIN ( "filter" , 			& PyFilter_Type );
	SETBUILTIN ( "float" , 				& PyFloat_Type );
	SETBUILTIN ( "frozenset" , 			& PyFrozenSet_Type );
	SETBUILTIN ( "property" , 			& PyProperty_Type );
	SETBUILTIN ( "int" , 				& PyLong_Type );
	SETBUILTIN ( "list" , 				& PyList_Type );
	SETBUILTIN ( "map" , 				& PyMap_Type );
	SETBUILTIN ( "object" , 			& PyBaseObject_Type );
	SETBUILTIN ( "range" , 				& PyRange_Type );
	SETBUILTIN ( "reversed" , 			& PyReversed_Type );
	SETBUILTIN ( "set" , 				& PySet_Type );
	SETBUILTIN ( "slice" , 				& PySlice_Type );
	SETBUILTIN ( "staticmethod" , 			& PyStaticMethod_Type );
	SETBUILTIN ( "str" , 				& PyUnicode_Type );
	SETBUILTIN ( "super" , 				& PySuper_Type );
	SETBUILTIN ( "tuple" , 				& PyTuple_Type );
	SETBUILTIN ( "type" , 				& PyType_Type );
	SETBUILTIN ( "zip" , 				& PyZip_Type );
	debug = PyBool_FromLong ( Py_OptimizeFlag == 0 );
	if ( PyDict_SetItemString ( dict , "__debug__" , debug ) < 0 ) {
		Py_DECREF ( debug );
		return NULL;
	}
	Py_DECREF ( debug );
	return mod;
#undef ADD_TO_ALL
#undef SETBUILTIN
}
````

All the standard built in functions of python are set in the following method defined in the file
bltinmodule.c line no 2677.

Let us now understand the implementation of the initialization of the sys module.

```C
PyObject *
_PySys_Init ( void)
{
	PyObject * m , * sysdict , * version_info;
	int res;
	m = PyModule_Create (& sysmodule );
	if ( m == NULL)
		return NULL;
	sysdict = PyModule_GetDict ( m );
#define SET_SYS_FROM_STRING_BORROW ( key , value )			\
	do { 						                \
		PyObject * v = ( value ); 		                \
		if ( v == NULL ) 				        \
			return NULL ;				        \
		res = PyDict_SetItemString ( sysdict , key , v ); 	\
		if ( res < 0 ) { 				        \
			return NULL ;				        \
		} 							\
	} while ( 0)
#define SET_SYS_FROM_STRING ( key , value ) 				\
	do { 													\
		PyObject * v = ( value ); 	          	        \
		if ( v == NULL ) 					\
			return NULL ; 					\
		res = PyDict_SetItemString ( sysdict , key , v ); 	\
		Py_DECREF ( v ); 				        \
		if ( res < 0 ) { 					\
			return NULL ; 					\
		} 												\
	} while ( 0)

	/* Check that stdin is not a directory
	Using shell redirection , you can redirect stdin to a directory,
	crashing the Python interpreter . Catch this common mistake here
	and output a useful error message . Note that under MS Windows,
	the shell already prevents that . */

#if !defined(MS_WINDOWS)
	{
		struct _Py_stat_struct sb;
		if ( _Py_fstat_noraise ( fileno ( stdin ), & sb ) == 0 &&
			S_ISDIR ( sb . st_mode )) {
				/* There's nothing more we can do. */
				/* Py_FatalError() will core dump, so just exit. */
			PySys_WriteStderr ( "Python error: <stdin> is a directory, cannot continue\n" );
			exit ( EXIT_FAILURE );
		}
	}
#endif

	/* stdin/stdout/stderr are set in pylifecycle.c */

	SET_SYS_FROM_STRING_BORROW ( "__displayhook__", PyDict_GetItemString ( sysdict , "displayhook" ));
	
	SET_SYS_FROM_STRING_BORROW ( "__excepthook__", PyDict_GetItemString ( sysdict , "excepthook" ));
	
	SET_SYS_FROM_STRING ( "version", PyUnicode_FromString ( Py_GetVersion ()));
	
	SET_SYS_FROM_STRING ( "hexversion", PyLong_FromLong ( PY_VERSION_HEX ));
	
	SET_SYS_FROM_STRING ( "_mercurial", Py_BuildValue ( "(szz)" , "CPython" , _Py_hgidentifier () , _Py_hgversion ()));
	
	SET_SYS_FROM_STRING ( "dont_write_bytecode", PyBool_FromLong ( Py_DontWriteBytecodeFlag ));
	
	SET_SYS_FROM_STRING ( "api_version", PyLong_FromLong ( PYTHON_API_VERSION ));
	
	SET_SYS_FROM_STRING ( "copyright", PyUnicode_FromString ( Py_GetCopyright ()));
	
	SET_SYS_FROM_STRING ( "platform", PyUnicode_FromString ( Py_GetPlatform ()));
	
	SET_SYS_FROM_STRING ( "executable", PyUnicode_FromWideChar( Py_GetProgramFullPath (), - 1 ));
	
	SET_SYS_FROM_STRING ( "prefix", PyUnicode_FromWideChar ( Py_GetPrefix (), - 1 ));
	
	SET_SYS_FROM_STRING ( "exec_prefix", PyUnicode_FromWideChar ( Py_GetExecPrefix (), - 1 ));
	
	SET_SYS_FROM_STRING ( "base_prefix", PyUnicode_FromWideChar ( Py_GetPrefix (), - 1 ));
	
	SET_SYS_FROM_STRING ( "base_exec_prefix", PyUnicode_FromWideChar ( Py_GetExecPrefix (), - 1 ));
	
	SET_SYS_FROM_STRING ( "maxsize", PyLong_FromSsize_t ( PY_SSIZE_T_MAX ));
	
	SET_SYS_FROM_STRING ( "float_info", PyFloat_GetInfo ());
	
	SET_SYS_FROM_STRING ( "int_info", PyLong_GetInfo ());
	
	/* initialize hash_info */

	if ( Hash_InfoType . tp_name == NULL ) {
		if ( PyStructSequence_InitType2 (& Hash_InfoType , & hash_info_desc ) < 0)
			return NULL;
}
	SET_SYS_FROM_STRING ( "hash_info", get_hash_info ());
	
	SET_SYS_FROM_STRING ( "maxunicode", PyLong_FromLong ( 0x10FFFF ));

	SET_SYS_FROM_STRING ( "builtin_module_names", list_builtin_module_names ());

#if PY_BIG_ENDIAN
	SET_SYS_FROM_STRING ( "byteorder", PyUnicode_FromString ( "big" ));

#else
	SET_SYS_FROM_STRING ( "byteorder", PyUnicode_FromString ( "little" ));

#endif

#ifdef MS_COREDLL
	SET_SYS_FROM_STRING ( "dllhandle", PyLong_FromVoidPtr ( PyWin_DLLhModule ));

	SET_SYS_FROM_STRING ( "winver", PyUnicode_FromString ( PyWin_DLLVersionString ));

#endif

#ifdef ABIFLAGS
	SET_SYS_FROM_STRING ( "abiflags", PyUnicode_FromString ( ABIFLAGS ));

#endif
	if ( warnoptions == NULL ) {
		warnoptions = PyList_New ( 0 );
		if ( warnoptions == NULL)
			return NULL;
	}
	else {
		Py_INCREF ( warnoptions );
	}
	SET_SYS_FROM_STRING_BORROW ( "warnoptions" , warnoptions );
	
	SET_SYS_FROM_STRING_BORROW ( "_xoptions" , get_xoptions ());

	/* version_info */
	if ( VersionInfoType . tp_name == NULL ) {
		if ( PyStructSequence_InitType2 (& VersionInfoType, & version_info_desc ) < 0)
			return NULL;
	}
	
	version_info = make_version_info ();
	SET_SYS_FROM_STRING ( "version_info" , version_info );
	/* prevent user from creating new instances */
	VersionInfoType . tp_init = NULL;
	VersionInfoType . tp_new = NULL;
	res = PyDict_DelItemString ( VersionInfoType . tp_dict , "__new__" );
	if ( res < 0 && PyErr_ExceptionMatches ( PyExc_KeyError ))
		PyErr_Clear ();

	/* implementation */
	SET_SYS_FROM_STRING ( "implementation" , make_impl_info ( version_info));

	/* flags */
	if ( FlagsType . tp_name == 0 ) {
		if ( PyStructSequence_InitType2 (& FlagsType , & flags_desc ) < 0)
			return NULL;
	}
	
	SET_SYS_FROM_STRING ( "flags" , make_flags ());
	/* prevent user from creating new instances */
	FlagsType . tp_init = NULL;
	FlagsType . tp_new = NULL;
	res = PyDict_DelItemString ( FlagsType . tp_dict , "__new__" );
	if ( res < 0 && PyErr_ExceptionMatches ( PyExc_KeyError ))
		PyErr_Clear ();

#if defined(MS_WINDOWS)
	/* getwindowsversion */
	if ( WindowsVersionType . tp_name == 0)
		if ( PyStructSequence_InitType2 (& WindowsVersionType, & windows_version_desc ) < 0)
			return NULL;
	/* prevent user from creating new instances */
	WindowsVersionType . tp_init = NULL;
	WindowsVersionType . tp_new = NULL;
	res = PyDict_DelItemString ( WindowsVersionType . tp_dict , "__new__" );
	if ( res < 0 && PyErr_ExceptionMatches ( PyExc_KeyError ))
		PyErr_Clear ();
#endif

	/* float repr style: 0.03 (short) vs 0.029999999999999999 (legacy) */

#ifndef PY_NO_SHORT_FLOAT_REPR
	SET_SYS_FROM_STRING ( "float_repr_style", PyUnicode_FromString ( "short" ));

#else
	SET_SYS_FROM_STRING ( "float_repr_style", PyUnicode_FromString ( "legacy" ));

#endif

#ifdef WITH_THREAD
	SET_SYS_FROM_STRING ( "thread_info" , PyThread_GetInfo ());
#endif
		
	/* initialize asyncgen_hooks */
	if ( AsyncGenHooksType . tp_name == NULL ) {
		if ( PyStructSequence_InitType2( & AsyncGenHooksType , & asyncgen_hooks_desc ) < 0 ) {
			return NULL;
		}
	}

#undef SET_SYS_FROM_STRING
#undef SET_SYS_FROM_STRING_BORROW
	if ( PyErr_Occurred ())
		return NULL;
	return m;
}
```

The initialized module is stored in the interpreter state as sysdict

```C
interp -> sysdict = PyModule_GetDict ( sysmod );
```

Let us now understand an import with a from clause

Modify the test.py as follows

```C
from test1 import func

1 		0 LOAD_CONST 			0 ( 0)
		2 LOAD_CONST 			1 (( 'func' ,))
		4 IMPORT_NAME 			0 ( test1)
		6 IMPORT_FROM 			1 ( func)
		8 STORE_NAME 			1 ( func)
		10 POP_TOP
```

Let us understand the implementation of the opcode IMPORT_FROM

ceval.c line no 2871

```C
TARGET ( IMPORT_FROM ) {
				PyObject * name = GETITEM ( names , oparg );
				PyObject * from = TOP ();
				PyObject * res;
				res = import_from ( from , name );
				PUSH ( res );
				if ( res == NULL)
					goto error;
				DISPATCH ();
			}

static PyObject *
import_from ( PyObject * v , PyObject * name)
{
	PyObject * x;
	_Py_IDENTIFIER ( __name__ );
	PyObject * fullmodname , * pkgname;

	x = PyObject_GetAttr ( v , name );
	if ( x != NULL || ! PyErr_ExceptionMatches ( PyExc_AttributeError ))
		return x;
	/* Issue #17636: in case this failed because of a circular relative
		import , try to fallback on reading the module directly from sys . modules . */
	PyErr_Clear ();
	pkgname = _PyObject_GetAttrId ( v , & PyId___name__ );
	if ( pkgname == NULL ) {
		goto error;
	}
	fullmodname = PyUnicode_FromFormat ( "%U.%U" , pkgname , name );
	Py_DECREF ( pkgname );
	if ( fullmodname == NULL ) {
		return NULL;
	}
	x = PyDict_GetItem ( PyImport_GetModuleDict (), fullmodname );
	Py_DECREF ( fullmodname );
	if ( x == NULL ) {
		goto error;
	}
	Py_INCREF ( x );
	return x;
error:
	PyErr_Format ( PyExc_ImportError , "cannot import name %R" , name );
	return NULL;
}
```

We see that fetching an item from a module is as simple as fetching an item from the modules
dictionary.

Let us now understand the implementation of the import * functionality of python

```C
from test1 import *

1 		0 LOAD_CONST 			0 ( 0)
		2 LOAD_CONST 			1 (( '*' ,))
		4 IMPORT_NAME 			0 ( test1)
		6 IMPORT_STAR
```

Let us understand the implementation of **IMPORT_STAR**

```C
TARGET ( IMPORT_STAR ) {
				PyObject * from = POP (), * locals;
				int err;
				if ( PyFrame_FastToLocalsWithError ( f ) < 0)
					goto error;

				locals = f -> f_locas;
				if ( locals == NULL ) {
					PyErr_SetString ( PyExc_SystemError, "no locals found during 'import *'" );
					goto error;	
				}
				err = import_all_from ( locals , from );
				PyFrame_LocalsToFast ( f , 0 );
				Py_DECREF ( from );
				if ( err != 0)
					goto error;
				DISPATCH ();
			}

static int
import_all_from ( PyObject * locals , PyObject * v)
{
	_Py_IDENTIFIER ( __all__ );
	_Py_IDENTIFIER ( __dict__ );
	PyObject * all = _PyObject_GetAttrId ( v , & PyId___all__ );
	PyObject * dict , * name , * value;
	int skip_leading_underscores = 0;
	int pos , err;
	
	if ( all == NULL ) {
		if (! PyErr_ExceptionMatches ( PyExc_AttributeError ))
			return - 1 ; /* Unexpected error */
		PyErr_Clear ();
		dict = _PyObject_GetAttrId ( v , & PyId___dict__ );
		if ( dict == NULL ) {
			if (! PyErr_ExceptionMatches ( PyExc_AttributeError ))
				return - 1;
			PyErr_SetString ( PyExc_ImportError, "from-import-* object has no __dict__ and no __all__" );
			return - 1;
		}
		all = PyMapping_Keys ( dict );
		Py_DECREF ( dict );
		if ( all == NULL)
			return - 1;
		skip_leading_underscores = 1;
	}

	for ( pos = 0 , err = 0 ; ; pos ++) {
		name = PySequence_GetItem ( all , pos );
		if ( name == NULL ) {
			if (! PyErr_ExceptionMatches ( PyExc_IndexError ))
				err = - 1;
			else
				PyErr_Clear ();
			break;
		}
		if ( skip_leading_underscores && PyUnicode_Check ( name ) && PyUnicode_READY ( name ) != - 1 && PyUnicode_READ_CHAR ( name , 0 ) == '_')
		{
			Py_DECREF ( name );
			continue;
		}
		value = PyObject_GetAttr ( v , name );
		if ( value == NULL)
			err = - 1;
		else if ( PyDict_CheckExact ( locals ))
			err = PyDict_SetItem ( locals , name , value );
		else
			err = PyObject_SetItem ( locals , name , value );
		Py_DECREF ( name );
		Py_XDECREF ( value );
		if ( err != 0)
			break;
	}
	Py_DECREF ( all );
	return err;
}
```

We understand that all the elements in the modules dictionary are imported into the locals
dictionary of the function frame. Simple is’nt it.

That’s all we have about the import statements. Hope you enjoyed the chapter.
