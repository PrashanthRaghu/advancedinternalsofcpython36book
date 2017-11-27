<h1 align="center"> Chapter​ ​ 8 </h1>
<h2 align="center"> Internals​ ​ of​ ​ the​ ​ interpreter </h2>
Let​ ​ us​ ​ understand​ ​ the​ ​ structure​ ​ that​ ​ stores​ ​ the​ ​ global​ ​ state​ ​ of​ ​ the​ ​ interpreter

```c
typedef​​ ​​struct​​ ​_is​ ​{
​ ​​ ​​ ​​ ​​struct​​ ​_is​ ​​*​next​;​​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​​struct​​ ​_ts​ ​​*​tstate_head​;​​ ​​//​ ​ 2
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​modules;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​modules_by_index;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​sysdict;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​builtins;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​importlib;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​codec_search_path;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​codec_search_cache;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​codec_error_registry;
​ ​​ ​​ ​​ ​​int​​ ​codecs_initialized;
​ ​​ ​​ ​​ ​​int​​ ​fscodec_initialized;
#ifdef​​ ​HAVE_DLOPEN
​ ​​ ​​ ​​ ​​int​​ ​dlopenflags;
#endif
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​builtins_copy;
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​import_func;
​ ​​ ​​ ​​ ​​/*​ ​Initialized​ ​to​ ​PyEval_EvalFrameDefault().​ ​*/
​ ​​ ​​ ​​ ​​_PyFrameEvalFunction​​ ​eval_frame;
}​​ ​​PyInterpreterState;
```
**1. All​​ ​the​ ​interpreters​ ​​in​​ ​the​ ​same​ ​process​ ​space​ ​are​ ​arranged​ ​​as​​ ​a singly​ ​linked​ ​list.**

**2. The interpreter state is initialized by the function PyInterpreterState_New in the file pystate.c line no 69**

```C
PyInterpreterState​​ ​*
PyInterpreterState_New​(​void)
{
​ ​​ ​​ ​​ ​​PyInterpreterState​​ ​​*​interp​ ​​=​​ ​​(​PyInterpreterState​​ ​​*)
    PyMem_RawMalloc​(​sizeof​(​PyInterpreterState​));
​ ​​ ​​ ​​ ​​if​​ ​​(​interp​ ​​!= NULL​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​HEAD_INIT​();
#ifdef​​ ​WITH_THREAD
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​head_mutex​ ​​== NULL)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_FatalError​(​"Can't​ ​initialize​ ​threads​ ​for​ ​interpreter"​);
#endif
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​modules​ ​​= NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​modules_by_index​ ​​= NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​sysdict​ ​​=​​ ​NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​builtins​ ​​=​​ ​NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​builtins_copy​ ​​= NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​tstate_head​ ​​=​​ ​NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​codec_search_path​ ​​=​​ ​NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​codec_search_cache​ ​​= NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​codec_error_registry​ ​​=​​ ​NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​codecs_initialized​ ​​=​​ ​​0;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​fscodec_initialized​ ​​=​​ ​​0;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​importlib​ ​​= NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​import_func​ ​​= NULL;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​eval_frame​ ​​=​​ ​​_PyEval_EvalFrameDefault;
#ifdef​​ ​HAVE_DLOPEN
#if​ ​HAVE_DECL_RTLD_NOW
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​dlopenflags​ ​​=​​ ​RTLD_NOW;
#else
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​dlopenflags​ ​​=​​ ​RTLD_LAZY;
#endif
#endif​​ ​​ ​
        HEAD_LOCK​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp​->​next​​ ​​=​​ ​interp_head;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​interp_head​ ​​=​​ ​interp;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​HEAD_UNLOCK​();
​ ​​ ​​ ​​ ​}

​ ​​ ​​ ​​ ​​return​​ ​interp;
}
```
**Topic GIL and the interpreter** 

**Implementation of the GIL details from ceval_gil.h**

```c
​ ​/*​ ​​ ​​Notes​​ ​about​ ​the​ ​implementation:
​ ​​ ​​ ​​-​​ ​​The​​ ​GIL​ ​​is​​ ​just​ ​a​ ​​boolean​​ ​variable​ ​​(​gil_locked​)​​ ​whose​ ​access​ ​​is protected
​ ​​ ​​ ​​ ​​ ​​by​​ ​a​ ​mutex​ ​​(​gil_mutex​),​​ ​​and​​ ​whose​ ​changes​ ​are​ ​signalled​ ​​by​​ a​ ​condition
​ ​​ ​​ ​​ ​​ ​variable​ ​​(​gil_cond​).​​ ​gil_mutex​ ​​is​​ ​taken​ ​​for​​ ​​short​​ ​periods​ of​ ​time, ​​and​​ 
     ​therefore​ ​mostly​ ​uncontended.
     
​ ​​ ​​ ​​-​​ ​​In​​ ​the​ ​GIL​-​holding​ ​thread​,​​ ​the​ ​main​ ​loop​ ​​(​PyEval_EvalFrameEx​)​​ ​must​ ​be
​ ​​ ​​ ​​ ​​ ​able​ ​to​ ​release​ ​the​ ​GIL​ ​on​ ​demand​ ​​by​​ ​another​ ​thread​.​​ ​A​ ​​volatile boolean
​ ​​ ​​ ​​ ​​ ​variable​ ​​(​gil_drop_request​)​​ ​​is​​ ​used​ ​​for​​ ​that​ ​purpose​,​​ ​which​ ​​is​​ ​​checked
​ ​​ ​​ ​​ ​​ ​at​ ​every​ ​turn​ ​of​ ​the​ ​​eval​​ ​loop​.​​ ​​That​​ ​variable​ ​​is​​ ​​set​​ ​after​ ​a​ ​wait​ ​of
​ ​​ ​​ ​​ ​​ ​​`interval`​​ ​microseconds​ ​on​ ​​`gil_cond`​​ ​has​ ​timed​ ​​out.

​ ​​ ​​ ​​ ​​ ​​[​Actually​,​​ ​another​ ​​volatile​​ ​​boolean​​ ​variable​ ​​(​eval_breaker​)​​ ​​is​​ ​used
​ ​​​ ​​​ ​​ ​​ ​which​ ​​ORs​​ ​several​ ​conditions​ ​​into​​ ​one​.​​ ​​Volatile​​ ​booleans​ ​are
​ ​​​​ ​​ ​​ ​​ ​sufficient​ ​​as​​ ​inter​-​thread​ ​signalling​ ​means​ ​since​ ​​Python​​ ​​is​​ ​run
​​​ ​​ ​​ ​​ ​​ ​on​ ​cache​-​coherent​ ​architectures​ ​only​.]

​ ​​ ​​ ​​-​​ ​A​ ​thread​ ​wanting​ ​to​ ​take​ ​the​ ​GIL​ ​will​ ​first​ ​let​ ​​pass​​ ​a​ ​given​ ​amount​ ​of
​ ​​ ​​ ​​ ​​ ​time​ ​​(​`interval`​​ ​microseconds​)​​ ​before​ ​setting​ ​gil_drop_request​.​​ ​​This
​ ​​ ​​ ​​ ​​ ​encourages​ ​a​ ​​defined​​ ​switching​ ​period​,​​ ​but​ ​doesn​'t​ ​enforce​ ​it​ ​since
​ ​​ ​​ ​​ ​​ ​opcodes​ ​can​ ​take​ ​an​ ​arbitrary​ ​time​ ​to​ ​execute.

​ ​​ ​​ ​​ ​​ ​​The​​ ​​`interval`​​ ​value​ ​​is​​ ​available​ ​​for​​ ​the​ ​user​ ​to​ ​read​ ​​and​​ ​modify
​ ​​ ​​ ​​ ​​ ​​using​​ ​the​ ​​Python​​ ​API​ ​​`sys.{get,set}switchinterval()`.

​ ​​ ​​ ​​-​​ ​​When​​ ​a​ ​thread​ ​releases​ ​the​ ​GIL​ ​​and​​ ​gil_drop_request​ ​​is​​ ​​set​,​​ ​that thread
​ ​​ ​​ ​​ ​​ ​ensures​ ​that​ ​another​ ​GIL​-​awaiting​ ​thread​ ​gets​ ​scheduled.
​ ​​ ​​ ​​ ​​ ​​It​​ ​does​ ​so​ ​​by​​ ​waiting​ ​on​ ​a​ ​condition​ ​variable​ ​​(​switch_cond​)​​ ​​until
​ ​​ ​​ ​​ ​​ ​the​ ​value​ ​of​ ​gil_last_holder​ ​​is​​ ​changed​ ​to​ ​something​ ​​else​​ ​than​ ​its
​ ​​ ​​ ​​ ​​ ​own​ ​thread​ ​state​ ​pointer​,​​ ​indicating​ ​that​ ​another​ ​thread​ ​was​ ​able​ ​to
​ ​​ ​​ ​​ ​​ ​take​ ​the​ ​GIL.

​ ​​ ​​ ​​ ​​ ​​This​​ ​​is​​ ​meant​ ​to​ ​prohibit​ ​the​ ​latency​-​adverse​ ​behaviour​ ​on​ ​multi​-​core
​ ​​ ​​ ​​ ​​ ​machines​ ​​where​​ ​one​ ​thread​ ​would​ ​speculatively​ ​release​ ​the​ ​GIL​,​​ ​but still
​ ​​ ​​ ​​ ​​ ​run​ ​​and​​ ​​end​​ ​up​ ​being​ ​the​ ​first​ ​to​ ​re​-​acquire​ ​it​,​​ ​making​ ​the "timeslices"
​ ​​ ​​ ​​ ​​ ​much​ ​longer​ ​than​ ​expected.
​ ​​ ​​ ​​ ​​ ​​(​Note​:​​ ​​this​​ ​mechanism​ ​​is​​ ​enabled​ ​​with​​ ​FORCE_SWITCHING​ ​above)

*/
```
**Finally let us all look at what the magical GIL really is**
```c
static​​ ​​_Py_atomic_int​​ ​gil_locked​ ​​=​​ ​​{-​ 1 ​};
```
**The mutex that protects the gil and the conditional variable that other threads wait on the gil**

```c
static​​ ​COND_T​ ​gil_cond;
static​​ ​MUTEX_T​ ​gil_mutex;
```
**The thread that holds the GIL wais for the next thread to take the GIL using the switch_cond**

```c
#ifdef​​ ​FORCE_SWITCHING
/*​ ​This​ ​condition​ ​variable​ ​helps​ ​the​ ​GIL-releasing​ ​thread​ ​wait​ ​for
​ ​​ ​​ ​a​ ​GIL​-​awaiting​ ​thread​ ​to​ ​be​ ​scheduled​ ​​and​​ ​take​ ​the​ ​GIL​.​​ ​​*/
static​​ ​COND_T​ ​switch_cond;
static​​ ​MUTEX_T​ ​switch_mutex;
#endif
```
**Creation of the GIL**

**Ceval_gil.h line no 135**

```c
static​​ ​​void​​ ​create_gil​(​void)
{
​ ​​ ​​ ​​ ​MUTEX_INIT​(​gil_mutex​);​​ ​​//​ ​ 1
#ifdef​​ ​FORCE_SWITCHING
​ ​​ ​​ ​​ ​MUTEX_INIT​(​switch_mutex​);​​ ​​//​ ​ 2
#endif
​ ​​ ​​ ​​ ​COND_INIT​(​gil_cond​);​​ ​​//​ ​ 3
#ifdef​​ ​FORCE_SWITCHING
​ ​​ ​​ ​​ ​COND_INIT​(​switch_cond​);​​ ​​//​ ​ 4
#endif
​ ​​ ​​ ​​ ​​_Py_atomic_store_relaxed​(&​gil_last_holder​,​​ ​​ 0 ​);
​ ​​ ​​ ​​ ​​_Py_ANNOTATE_RWLOCK_CREATE​(&​gil_locked​);
​ ​​ ​​ ​​ ​​_Py_atomic_store_explicit​(&​gil_locked​,​​ ​​ 0 ​,​​ ​​_Py_memory_order_release​);​​ ​​//5
}
*)​​ ​​#define​​ ​​PyMUTEX_INIT​(​mut​)​​ ​​ ​​ ​​ ​​ ​​ ​​ ​pthread_mutex_init​((​mut​), NULL)
*)​​ ​​#define​​ ​​PyCOND_INIT​(​cond​)​​ ​​ ​​ ​​ ​​ ​​ ​​ ​pthread_cond_init​((​cond​), NULL)

1. Initialization​​ ​of​ ​the​ ​gil_mutex
2. Initialization​​ ​of​ ​the​ ​switch_mutex
3. Initialization​​ ​of​ ​the​ ​gil​ ​conditional​ ​variable
4. Initialization​​ ​of​ ​the​ ​​switch​​ ​conditional​ ​variable
```
**When is the GIL created?**

**ceval.c line no 226**

```c
void PyEval_InitThreads​(​void)
{
​ ​​ ​​ ​​ ​​if​​ ​​(​gil_created​())
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return;
​ ​​ ​​ ​​ ​create_gil​();
​ ​​ ​​ ​​ ​take_gil​(​PyThreadState_GET​());
​ ​​ ​​ ​​ ​main_thread​ ​​=​​ ​​PyThread_get_thread_ident​();
​ ​​ ​​ ​​ ​​if​​ ​​(!​pending_lock)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​pending_lock​ ​​=​​ ​​PyThread_allocate_lock​();
}
```
**The** ​ ​ **GIL** ​ ​ **can** ​ ​ **be** ​ ​ **initialized** ​ ​ **from** ​ ​ **multiple** ​ ​ **places** ​ ​ **but** ​ ​ **one** ​ ​ **such** ​ ​ **location** ​ ​ **is** ​ ​ **_threadmodule.c** ​ ​ **line no** ​ ​ **1031**

```c
static​​ ​​PyObject​​ ​*
thread_PyThread_start_new_thread​(​PyObject​​ ​​*​self​,​​ ​​PyObject​​ ​​*​fargs)
{
​ ​​ ​​ ​​ ​​PyObject​​ ​​*​func​,​​ ​​*​args​,​​ ​​*​keyw​ ​​=​​ ​NULL;
​ ​​ ​​ ​​ ​​struct​​ ​bootstate​ ​​*​boot;
​ ​​ ​​ ​​ ​​long​​ ​ident;
​ ​​ ​​ ​​ ​​if​​ ​​(!​PyArg_UnpackTuple​(​fargs​,​​ ​​"start_new_thread"​,​​ ​​ 2 ​,​​ ​​3,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​&​func​,​​ ​​&​args​,​​ ​​&​keyw​))
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return NULL;
​ ​​ ​​ ​​ ​​if​​ ​​(!​PyCallable_Check​(​func​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyErr_SetString​(​PyExc_TypeError,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​"first​ ​arg​ ​must​ ​be​ ​callable"​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return NULL;
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​if​​ ​​(!​PyTuple_Check​(​args​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyErr_SetString​(​PyExc_TypeError,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​"2nd​ ​arg​ ​must​ ​be​ ​a​ ​tuple"​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return NULL;
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​if​​ ​​(​keyw​ ​​!=​​ ​NULL​ ​​&&​​ ​​!​PyDict_Check​(​keyw​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyErr_SetString​(​PyExc_TypeError,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​"optional​ ​3rd​ ​arg​ ​must​ ​be​ ​a​ ​dictionary"​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return NULL;
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​boot​ ​​=​​ ​​PyMem_NEW​(​struct​​ ​bootstate​,​​ ​​ 1 ​);
​ ​​ ​​ ​​ ​​if​​ ​​(​boot​ ​​==​​ ​NULL)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return​​ ​​PyErr_NoMemory​();
​ ​​ ​​ ​​ ​boot​->​interp​ ​​=​​ ​​PyThreadState_GET​()->​interp;
​ ​​ ​​ ​​ ​boot​->​func​ ​​=​​ ​func;
​ ​​ ​​ ​​ ​boot​->​args​ ​​=​​ ​args;
​ ​​ ​​ ​​ ​boot​->​keyw​ ​​=​​ ​keyw;
​ ​​ ​​ ​​ ​boot​->​tstate​ ​​=​​ ​​_PyThreadState_Prealloc​(​boot​->​interp​);
​ ​​ ​​ ​​ ​​if​​ ​​(​boot​->​tstate​ ​​==​​ ​NULL​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyMem_DEL​(​boot​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return​​ ​​PyErr_NoMemory​();
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​Py_INCREF​(​func​);
​ ​​ ​​ ​​ ​​Py_INCREF​(​args​);
​ ​​ ​​ ​​ ​​Py_XINCREF​(​keyw​);
​ ​​ ​​ ​​ ​​PyEval_InitThreads​();​​ ​​/*​ ​Start​ ​the​ ​interpreter's​ ​thread-awareness​ ​*/​​ ​​//1
​ ​​ ​​ ​​ ​ident​ ​​=​​ ​​PyThread_start_new_thread​(​t_bootstrap​,​​ ​​(​void​*)​​ ​boot​);
​ ​​ ​​ ​​ ​​if​​ ​​(​ident​ ​​==​​ ​​-​ 1 ​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyErr_SetString​(​ThreadError​,​​ ​​"can't​ ​start​ ​new​ ​thread"​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_DECREF​(​func​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_DECREF​(​args​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_XDECREF​(​keyw​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyThreadState_Clear​(​boot​->​tstate​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyMem_DEL​(​boot​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​return NULL;
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​return​​ ​​PyLong_FromLong​(​ident​);
}

1. The​​ ​GIL​ ​maybe​ ​initialized
```
**How** ​ ​ **is** ​ ​ **the** ​ ​ **GIL** ​ ​ **taken?**

**Ceval_gil.h** ​ ​ **line** ​ ​ **no** ​ ​ **207**

```c
static​​ ​​void​​ ​take_gil​(​PyThreadState​​ ​​*​tstate)
{
​ ​​ ​​ ​​ ​​int​​ ​err;
​ ​​ ​​ ​​ ​​if​​ ​​(​tstate​ ​​==​​ ​NULL)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_FatalError​(​"take_gil:​ ​NULL​ ​tstate"​);
​ ​​ ​​ ​​ ​err​ ​​=​​ ​errno;
​ ​​ ​​ ​​ ​MUTEX_LOCK​(​gil_mutex​);​​ ​​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​​if​​ ​​(!​_Py_atomic_load_relaxed​(&​gil_locked​))
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​_ready;

​ ​​ ​​ ​​ ​​while​​ ​​(​_Py_atomic_load_relaxed​(&​gil_locked​))​​ ​​{​​ ​​//​ ​ 2
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​int​​ ​timed_out​ ​​=​​ ​​0;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​unsigned​​ ​​long​​ ​saved_switchnum;

​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​saved_switchnum​ ​​=​​ ​gil_switch_number;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​COND_TIMED_WAIT​(​gil_cond​,​​ ​gil_mutex​,​​ ​INTERVAL​,​​ ​timed_out​);​​ ​​//​ ​ 3
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​If​ ​we​ ​timed​ ​out​ ​and​ ​no​ ​switch​ ​occurred​ ​in​ ​the​ ​meantime,​ ​it​ ​is time
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​to​ ​ask​ ​the​ ​GIL​-​holding​ ​thread​ ​to​ ​drop​ ​it​.​​ ​​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​timed_out​ ​​&&
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​_Py_atomic_load_relaxed​(&​gil_locked​)​​ ​​&&
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​gil_switch_number​ ​​==​​ ​saved_switchnum​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​SET_GIL_DROP_REQUEST​();​​ ​​//​ ​ 4
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​}
_ready:
#ifdef​​ ​FORCE_SWITCHING
​ ​​ ​​ ​​ ​​/*​ ​This​ ​mutex​ ​must​ ​be​ ​taken​ ​before​ ​modifying​ ​gil_last_holder​ ​(see
    drop_gil()).​ ​*/
​ ​​ ​​ ​​ ​MUTEX_LOCK​(​switch_mutex​);​​ ​​ ​​//​ ​ 5
#endif
​ ​​ ​​ ​​ ​​/*​ ​We​ ​now​ ​hold​ ​the​ ​GIL​ ​*/
​ ​​ ​​ ​​ ​​_Py_atomic_store_relaxed​(&​gil_locked​,​​ ​​ 1 ​);​​ ​​//​ ​ 6
​ ​​ ​​ ​​ ​​_Py_ANNOTATE_RWLOCK_ACQUIRED​(&​gil_locked​,​​ ​​/*is_write=*/​ 1 ​);

​ ​​ ​​ ​​ ​​if​​ ​​(​tstate​ ​​!= (​PyThreadState​*)​_Py_atomic_load_relaxed​(&​gil_last_holder​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​_Py_atomic_store_relaxed​(&​gil_last_holder​,​​ ​​(​uintptr_t​)​tstate​);​​ ​​//​ ​ 7
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​++​gil_switch_number​;​​ ​​//​ ​ 8
​ ​​ ​​ ​​ ​}

#ifdef​​ ​FORCE_SWITCHING
​ ​​ ​​ ​​ ​COND_SIGNAL​(​switch_cond​);​​ ​​//​ ​ 9
​ ​​ ​​ ​​ ​MUTEX_UNLOCK​(​switch_mutex​);​​ ​​//​ ​ 10
#endif
​ ​​ ​​ ​​ ​​if​​ ​​(​_Py_atomic_load_relaxed​(&​gil_drop_request​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​RESET_GIL_DROP_REQUEST​();
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​if​​ ​​(​tstate​->​async_exc​ ​​!=​​ ​NULL​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​_PyEval_SignalAsyncExc​();​​ ​​//​ ​ 11
​ ​​ ​​ ​​ ​}

​ ​​ ​​ ​​ ​MUTEX_UNLOCK​(​gil_mutex​);​​ ​​//​ ​ 12
​ ​​ ​​ ​​ ​errno​ ​​=​​ ​err;
}

1. Acquire​​ ​the​ ​​lock​​ ​on​ ​the​ ​​Mutex​​ ​gil_mutex​ ​to​ ​obtain​ ​access​ ​to​ ​the​ ​code.
2. Wait​​ ​​until​​ ​the​ ​​lock​​ ​​is​​ ​held​ ​​by​​ ​another​ ​thread.
3. Wait​​ ​​until​​ ​the​ ​thread​ ​signals​ ​on​ ​the​ ​condition​ ​gil_cond​ ​on​ ​a​ ​​fixed
    interval.
4. Ask​​ ​the​ ​thread​ ​to​ ​drop​ ​the​ ​gil
5. Acquire​​ ​the​ ​​switch​​ ​mutex
6. Acquire​​ ​the​ ​GIL
7. Set​​ ​the​ ​current​ ​thread​ ​holding​ ​the​ ​GIL
8. Increment​​ ​the​ ​​no​​ ​of​ ​times​ ​the​ ​GIL​ ​has​ ​switched
9. Signal​​ ​the​ ​previous​ ​thread​ ​that​ ​a​ ​​new​​ ​thread​ ​has​ ​acquired​ ​the​ ​thread
10.Unlock​​ ​the​ ​​switch​​ ​mutex
11.Signal​​ ​the​ ​asynchronous​ ​requests​ ​to​ ​execute​ ​​if​​ ​the​ ​thread​ ​allows
    asynchronous​ ​execution
12.Unlock​​ ​the​ ​GIL​ ​mutex
```
**When** ​ ​ **is** ​ ​ **the** ​ ​ **GIL** ​ ​ **taken?**

**ceval.c** ​ ​ **1108**

```c
...
for​​ ​​(;;)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​assert​(​stack_pointer​ ​​>=​​ ​f​->​f_valuestack​);​​ ​​/*​ ​else​ ​underflow​ ​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​assert​(​STACK_LEVEL​()​​ ​​<=​​ ​co​->​co_stacksize​);​​ ​​ ​​/*​ ​else​ ​overflow​ ​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​assert​(!​PyErr_Occurred​());

​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​Do​ ​periodic​ ​things.​ ​​ ​Doing​ ​this​ ​every​ ​time​ ​through
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​the​ ​loop​ ​would​ ​add​ ​too​ ​much​ ​overhead​,​​ ​so​ ​we​ ​​do​​ ​it
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​only​ ​every​ ​​Nth​​ ​instruction​.​​ ​​ ​​We​​ ​also​ ​​do​​ ​it​ ​​if
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​``​pendingcalls_to_do​''​​ ​​is​​ ​​set​,​​ ​i​.​e​.​​ ​​when​​ ​an​ ​asynchronous
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​event​​ ​needs​ ​attention​ ​​(​e​.​g​.​​ ​a​ ​signal​ ​handler​ ​​or
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​async​ ​I​/​O​ ​handler​);​​ ​see​ ​​Py_AddPendingCall​()​​ ​​and
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_MakePendingCalls​()​​ ​above​.​​ ​​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​_Py_atomic_load_relaxed​(&​eval_breaker​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​_Py_OPCODE​(*​next_instr​)​​ ​​==​​ ​SETUP_FINALLY​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​Make​ ​the​ ​last​ ​opcode​ ​before
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​a​ ​​try​:​​ ​​finally​:​​ ​block​ ​uninterruptible​.​​ ​​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​fast_next_opcode;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​_Py_atomic_load_relaxed​(&​pendingcalls_to_do​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​Py_MakePendingCalls​()​​ ​​<​​ ​​0)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​error;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
#ifdef​​ ​WITH_THREAD
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​_Py_atomic_load_relaxed​(&​gil_drop_request​))​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​Give​ ​another​ ​thread​ ​a​ ​chance​ ​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​PyThreadState_Swap(NULL)​​ ​​!=​​ ​tstate)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_FatalError​(​"ceval:​ ​tstate​ ​mix-up"​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​drop_gil​(​tstate​);​​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​Other​ ​threads​ ​may​ ​run​ ​now​ ​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​take_gil​(​tstate​);​​ ​​//​ ​ 2
...

1. When​​ ​another​ ​thread​ ​has​ ​​set​​ ​the​ ​drop​ ​request​ ​the​ ​thread​ ​drops​ ​the
    GIL.
2. The​​ ​current​ ​thread​ ​waits​ ​to​ ​acquire​ ​the​ ​GIL​ ​​while​​ ​the​ ​other​ ​thread
    starts​ ​it​'​s​ ​execution.
```
**How** ​ ​ **is** ​ ​ **the** ​ ​ **GIL** ​ ​ **dropped?**

**Ceval_gil.h** ​ ​ **line** ​ ​ **no** ​ ​ **172**

```c
static​​ ​​void​​ ​drop_gil​(​PyThreadState​​ ​​*​tstate)
{
​ ​​ ​​ ​​ ​​if​​ ​​(!​_Py_atomic_load_relaxed​(&​gil_locked​))
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_FatalError​(​"drop_gil:​ ​GIL​ ​is​ ​not​ ​locked"​);
​ ​​ ​​ ​​ ​​/*​ ​tstate​ ​is​ ​allowed​ ​to​ ​be​ ​NULL​ ​(early​ ​interpreter​ ​init)​ ​*/
​ ​​ ​​ ​​ ​​if​​ ​​(​tstate​ ​​!=​​ ​NULL​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​Sub-interpreter​ ​support:​ ​threads​ ​might​ ​have​ ​been​ ​switched
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​under​ ​​our​​ ​feet​ ​​using​​ ​​PyThreadState_Swap​().​​ ​​Fix​​ ​the​ ​GIL​ ​​last
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​holder​ ​variable​ ​so​ ​that​ ​​our​​ ​heuristics​ ​work​.​​ ​​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​_Py_atomic_store_relaxed​(&​gil_last_holder​,​​ ​​(​uintptr_t​)​tstate​);
​ ​​ ​​ ​​ ​}

​ ​​ ​​ ​​ ​MUTEX_LOCK​(​gil_mutex​);​​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​​_Py_ANNOTATE_RWLOCK_RELEASED​(&​gil_locked​,​​ ​​/*is_write=*/​ 1 ​);
​ ​​ ​​ ​​ ​​_Py_atomic_store_relaxed​(&​gil_locked​,​​ ​​ 0 ​);​​ ​​//​ ​ 2
​ ​​ ​​ ​​ ​COND_SIGNAL​(​gil_cond​);​​ ​​//​ ​ 3
​ ​​ ​​ ​​ ​MUTEX_UNLOCK​(​gil_mutex​);​​ ​​//​ ​ 4
#ifdef​​ ​FORCE_SWITCHING
​ ​​ ​​ ​​ ​​if​​ ​​(​_Py_atomic_load_relaxed​(&​gil_drop_request​)​​ ​​&&​​ ​tstate​ ​​!=​​ ​NULL​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​MUTEX_LOCK​(​switch_mutex​);​​ ​​//​ ​ 5
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​Not​ ​switched​ ​yet​ ​=>​ ​wait​ ​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​((​PyThreadState​*)​_Py_atomic_load_relaxed​(&​gil_last_holder​)​​ ​​== tstate​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​RESET_GIL_DROP_REQUEST​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​NOTE:​ ​if​ ​COND_WAIT​ ​does​ ​not​ ​atomically​ ​start​ ​waiting​ ​when
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​releasing​ ​the​ ​mutex​,​​ ​another​ ​thread​ ​can​ ​run​ ​through​,​​ ​take
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​the​ ​GIL​ ​​and​​ ​drop​ ​it​ ​again​,​​ ​​and​​ ​reset​ ​the​ ​condition
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​before​ ​we​ ​even​ ​had​ ​a​ ​chance​ ​to​ ​wait​ ​​for​​ ​it​.​​ ​​*/
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​COND_WAIT​(​switch_cond​,​​ ​switch_mutex​);​​ ​​//​ ​ 6
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​MUTEX_UNLOCK​(​switch_mutex​);​​ ​​//​ ​ 7
​ ​​ ​​ ​​ ​}
#endif
}

1. Lock​​ ​the​ ​gil_mutex
2. Drop​​ ​the​ ​GIL
3. Signal​​ ​to​ ​all​ ​threads​ ​waiting​ ​on​ ​the​ ​condition​ ​that​ ​the​ ​GIL​ ​has​ ​been
    released
4. Unlock​​ ​the​ ​mutex​ ​gil_mutex
5. Lock​​ ​the​ ​​switch​​ ​mutex
6. The​​ ​current​ ​thread​ ​waits​ ​​for​​ ​a​ ​thread​ ​to​ ​take​ ​the​ ​GIL​ ​indefinitely
7. Unlock​​ ​the​ ​​switch​​ ​mutex​ ​after​ ​a​ ​thread​ ​has​ ​taken​ ​the​ ​GIL.
```
### Topic ​ ​​ ​ Instruction ​ ​ dispatching ​ ​ within ​ ​ the ​ ​ interpreter ​ ​ loop

**We** ​ ​ **understand** ​ ​ **that** ​ ​ **the** ​ ​ **interpreter** ​ ​ **is** ​ ​ **a** ​ ​ **giant** ​ ​ **loop** ​ ​ **executing** ​ ​ **opcodes** ​ ​ **one** ​ ​ **after** ​ ​ **the** ​ ​ **other.** ​ ​ **But in** ​ ​ **order** ​ ​ **to** ​ ​ **ensure** ​ ​ **faster** ​ ​ **instruction** ​ ​ **of** ​ ​ **opcodes** ​ ​ **python** ​ ​ **interpreter** ​ ​ **provides** ​ ​ **a** ​ ​ **loop** ​ ​ **breaker for** ​ ​ **faster** ​ ​ **instruction** ​ ​ **dispatching.**

**The** ​ ​ **instruction** ​ ​ **dispatching** ​ ​ **is** ​ ​ **handled** ​ ​ **by** ​ ​ **two** ​ ​ **macros** ​ ​ **and** ​ ​ **a** ​ ​ **boolean** ​ ​ **variable**

```c
static​​ ​​_Py_atomic_int​​ ​eval_breaker​ ​​=​​ ​​{​ 0 ​};
```
**When** ​ ​ **eval_breaker** ​ ​ **is** ​ ​ **set** ​ ​ **to** ​ ​ **1** ​ ​ **execute** ​ ​ **instructions** ​ ​ **the** ​ ​ **faster** ​ ​ **way.**

**The** ​ ​ **macros** ​ ​ **that** ​ ​ **handle** ​ ​ **the** ​ ​ **eval** ​ ​ **breaker.**

```c
#define​​ ​DISPATCH​()​​ ​\
​ ​​ ​​ ​​ ​​{​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(!​_Py_atomic_load_relaxed​(&​eval_breaker​))​​ ​​{​​ ​​ ​​ ​​ ​​ ​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​FAST_DISPATCH​();​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​}​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​continue​;​​ ​\
​ ​​ ​​ ​​ ​}
#ifdef​​ ​LLTRACE
#define​​ ​FAST_DISPATCH​()​​ ​\
​ ​​ ​​ ​​ ​​{​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(!​lltrace​ ​​&&​​ ​​!​_Py_TracingPossible​​ ​​&&​​ ​​!​PyDTrace_LINE_ENABLED​())​​ ​​{ \
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​f​->​f_lasti​ ​​=​​ ​INSTR_OFFSET​();​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​NEXTOPARG​();​​ ​\​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​​*​opcode_targets​[​opcode​];​​ ​\​ ​​//​ ​ 2
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​}​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​fast_next_opcode​;​​ ​\
​ ​​ ​​ ​​ ​}
#define​​ ​NEXTOPARG​()​​ ​​ ​​do​​ ​​{​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​_Py_CODEUNIT​​ ​word​ ​​=​​ ​​*​next_instr​;​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​opcode​ ​​=​​ ​​_Py_OPCODE​(​word​);​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​oparg​ ​​=​​ ​​_Py_OPARG​(​word​);​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​next_instr​++;​​ ​\
​ ​​ ​​ ​​ ​​}​​ ​​while​​ ​​(​0)
```
**Observation** ​ ​ **1**


**1. Fetch** ​ ​ **the** ​ ​ **next** ​ ​ **opcode** ​ ​ **and** ​ ​ **op** ​ ​ **argument**

**2. Go** ​ ​ **to** ​ ​ **the** ​ ​ **precomputed** ​ ​ **opcode** ​ ​ **target** ​ ​ **within** ​ ​ **the** ​ ​ **loop**


**All** ​ ​ **the** ​ ​ **opcode** ​ ​ **targets** ​ ​ **are** ​ ​ **defined** ​ ​ **within** ​ ​ **the** ​ ​ **file** ​ ​ **opcode_targets.h**

```c
static​​ ​​void​​ ​​*​opcode_targets​[​ 256 ​]​​ ​​=​​ ​{
​ ​​ ​​ ​​ ​​&&​_unknown_opcode,
​ ​​ ​​ ​​ ​​&&​TARGET_POP_TOP,
​ ​​ ​​ ​​ ​​&&​TARGET_ROT_TWO,
​ ​​ ​​ ​​ ​​&&​TARGET_ROT_THREE,
​ ​​ ​​ ​​ ​​&&​TARGET_DUP_TOP,
​ ​​ ​​ ​​ ​​&&​TARGET_DUP_TOP_TWO,
​ ​​ ​​ ​​ ​​&&​_unknown_opcode,
​ ​​ ​​ ​​ ​​&&​_unknown_opcode,
​ ​​ ​​ ​​ ​​&&​_unknown_opcode,
​ ​​ ​​ ​​ ​​&&​TARGET_NOP,
​ ​​ ​​ ​​ ​​&&​TARGET_UNARY_POSITIVE,
​ ​​ ​​ ​​ ​​&&​TARGET_UNARY_NEGATIVE,
​ ​​ ​​ ​​ ​​&&​TARGET_UNARY_NOT,
​ ​​&&​_unknown_opcode,
​ ​​&&​_unknown_opcode,
​ ​​&&​TARGET_UNARY_INVERT,
​ ​​&&​TARGET_BINARY_MATRIX_MULTIPLY,
​ ​​&&​TARGET_INPLACE_MATRIX_MULTIPLY,
...
```
**How** ​ ​ **are** ​ ​ **the** ​ ​ **targets** ​ ​ **generated?**


```c
#define​​ ​TARGET​(​op​)​​ ​\
​ ​​ ​​ ​​ ​TARGET_​##op:​ ​\
​ ​​ ​​ ​​ ​​case​​ ​op:

Generates​​ ​a​ ​jump​ ​target​ ​​and​​ ​a​ ​​switch​​ ​condition​ ​​and​​ ​hence​ ​works​ ​​for​​ ​both​ ​​in
the​ ​loop​ ​mode​ ​​as​​ ​well​ ​​as​​ ​the​ ​fast​ ​loop​ ​breaker​ ​mode.
```
**Example** ​ ​ **of** ​ ​ **an** ​ ​ **opcode** ​ ​ **target** ​ ​ **generation** ​ ​ **for** ​ ​ **opcode** ​ ​ **LOAD_FAST**
```c
 TARGET​(​LOAD_FAST​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​value​ ​​=​​ ​GETLOCAL​(​oparg​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​value​ ​​== NULL​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​format_exc_check_arg​(​PyExc_UnboundLocalError,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​UNBOUNDLOCAL_ERROR_MSG,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyTuple_GetItem​(​co​->​co_varnames​, oparg​));
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​error;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_INCREF​(​value​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​PUSH​(​value​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​FAST_DISPATCH​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
```
### Topic ​ ​ Opcode ​ ​ prediction ​ ​ in ​ ​ the ​ ​ evaluation ​ ​ loop

**P.S** ​ ​ **This** ​ ​ **execution** ​ ​ **mode** ​ ​ **is** ​ ​ **used** ​ ​ **when** ​ ​ **the** ​ ​ **Computed** ​ ​ **Goto’s** ​ ​ **using** ​ ​ **eval** ​ ​ **breaker** ​ ​ **is** ​ ​ **not** ​ ​ **used.**

**Let** ​ ​ **us** ​ ​ **understand** ​ ​ **the** ​ ​ **opcode** ​ ​ **implementation** ​ ​ **for** ​ ​ **COMPARE_OP**
```c
 TARGET​(​COMPARE_OP​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​right​ ​​=​​ ​POP​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​left​ ​​=​​ ​TOP​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​res​ ​​=​​ ​cmp_outcome​(​oparg​,​​ ​left​,​​ ​right​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_DECREF​(​left​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_DECREF​(​right​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​SET_TOP​(​res​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​res​ ​​==​​ ​NULL)
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​error;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​PREDICT​(​POP_JUMP_IF_FALSE​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​PREDICT​(​POP_JUMP_IF_TRUE​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​DISPATCH​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
```
**The** ​ ​ **opcode** ​ ​ **after** ​ ​ **a** ​ ​ **compare** ​ ​ **OP** ​ ​ **is** ​ ​ **either** ​ ​ **a** ​ ​ **POP_JUMP_IF_FALSE** ​ ​ **or** ​ ​ **POP_JUMP_IF_TRUE** ​ ​ **let** ​ ​ **us hence** ​ ​ **observe** ​ ​ **the** ​ ​ **code** ​ ​ **flow** ​ ​ **for** ​ ​ **these** ​ ​ **macros.**


```c
#define​​ ​PREDICT​(​op​)​​ ​\
​ ​​ ​​ ​​ ​​do​{​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​_Py_CODEUNIT​​ ​word​ ​​=​​ ​​*​next_instr​;​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​opcode​ ​​=​​ ​​_Py_OPCODE​(​word​);​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​if​​ ​​(​opcode​ ​​==​​ ​op​){​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​oparg​ ​​=​​ ​​_Py_OPARG​(​word​);​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​next_instr​++;​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​goto​​ ​PRED_​##op;​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​}​​ ​\
​ ​​ ​​ ​​ ​​}​​ ​​while​(​0)
#define​​ ​PREDICTED​(​op​)​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​PRED_​##op:
```
**Example:**
```c
PREDICTED​(​LOAD_CONST​);
```
### Topic ​ ​ Important ​ ​ variables ​ ​ initialization ​ ​ within ​ ​ the ​ ​ interpreter ​ ​ loop

**ceval.c** ​ ​ **1045**

```c
​ ​​ ​​ ​​ ​co​ ​​=​​ ​f​->​f_code​;​​ ​​ ​​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​names​ ​​=​​ ​co​->​co_names​;​​ ​​ ​​//​ ​ 2
​ ​​ ​​ ​​ ​consts​ ​​=​​ ​co​->​co_consts​;​​ ​​ ​​//​ ​ 3
​ ​​ ​​ ​​ ​fastlocals​ ​​=​​ ​f​->​f_localsplus​;​​ ​​//​ ​ 4
​ ​​ ​​ ​​ ​freevars​ ​​=​​ ​f​->​f_localsplus​ ​​+​​ ​co​->​co_nlocals;
​ ​​ ​​ ​​ ​​assert​(​PyBytes_Check​(​co​->​co_code​));
​ ​​ ​​ ​​ ​​assert​(​PyBytes_GET_SIZE​(​co​->​co_code​)​​ ​​<=​​ ​INT_MAX​);
​ ​​ ​​ ​​ ​​assert​(​PyBytes_GET_SIZE​(​co​->​co_code​)​​ ​​%​​ ​​sizeof​(​_Py_CODEUNIT​)​​ ​​==​​ ​​ 0 ​);
​ ​​ ​​ ​​ ​​assert​(​_Py_IS_ALIGNED​(​PyBytes_AS_STRING​(​co​->​co_code​), sizeof​(​_Py_CODEUNIT​)));
​ ​​ ​​ ​​ ​first_instr​ ​​=​​ ​​(​_Py_CODEUNIT​​ ​​*)​​ ​​PyBytes_AS_STRING​(​co​->​co_code​);​​ ​​//​ ​ 5
​ ​​ ​​ ​​ ​​/*
​ ​​ ​​ ​​ ​​ ​​ ​​ ​f​->​f_lasti​ ​refers​ ​to​ ​the​ ​index​ ​of​ ​the​ ​​last​​ ​instruction,
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​unless​​ ​it​'s​ ​-1​ ​in​ ​which​ ​case​ ​next_instr​ ​should​ ​be​ ​first_instr.
​ ​​ ​​ ​​ ​​ ​​ ​​ ​YIELD_FROM​ ​sets​ ​f_lasti​ ​to​ ​itself​,​​ ​​in​​ ​order​ ​to​ ​repeatedly​ ​​yield
​ ​​ ​​ ​​ ​​ ​​ ​​ ​multiple​ ​values.
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​When​​ ​the​ ​PREDICT​()​​ ​macros​ ​are​ ​enabled​,​​ ​some​ ​opcode​ ​pairs​ ​follow​ ​​in
​ ​​ ​​ ​​ ​​ ​​ ​​ ​direct​ ​succession​ ​without​ ​updating​ ​f​->​f_lasti​.​​ ​​ ​A​ ​successful
​ ​​ ​​ ​​ ​​ ​​ ​​ ​prediction​ ​effectively​ ​links​ ​the​ ​two​ ​codes​ ​together​ ​​as​​ ​​if​​ ​they
​ ​​ ​​ ​​ ​​ ​​ ​​ ​were​ ​a​ ​single​ ​​new​​ ​opcode​;​​ ​accordingly​,​f​->​f_lasti​ ​will​ ​point​ ​to
​ ​​ ​​ ​​ ​​ ​​ ​​ ​the​ ​first​ ​code​ ​​in​​ ​the​ ​pair​ ​​(​for​​ ​instance​,​​ ​GET_ITER​ ​followed​ ​​by
​ ​​ ​​ ​​ ​​ ​​ ​​ ​FOR_ITER​ ​​is​​ ​effectively​ ​a​ ​single​ ​opcode​ ​​and​​ ​f​->​f_lasti​ ​will​ ​point
​ ​​ ​​ ​​ ​​ ​​ ​​ ​to​ ​the​ ​beginning​ ​of​ ​the​ ​combined​ ​pair​.)
​ ​​ ​​ ​​ ​​*/
​ ​​ ​​ ​​ ​​assert​(​f​->​f_lasti​ ​​>=​​ ​​-​ 1 ​);
​ ​​ ​​ ​​ ​next_instr​ ​​=​​ ​first_instr​;
​ ​​ ​​ ​​ ​​if​​ ​​(​f​->​f_lasti​ ​​>=​​ ​​ 0 ​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​assert​(​f​->​f_lasti​ ​​%​​ ​​sizeof​(​_Py_CODEUNIT​)​​ ​​==​​ ​​ 0 ​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​next_instr​ ​​+=​​ ​f​->​f_lasti​ ​​/​​ ​​sizeof​(​_Py_CODEUNIT​)​​ ​​+​​ ​​1;
​ ​​ ​​ ​​ ​}
​ ​​ ​​ ​​ ​stack_pointer​ ​​=​​ ​f​->​f_stacktop​;​​ ​​//​ ​ 6
​ ​​ ​​ ​​ ​​assert​(​stack_pointer​ ​​!=​​ ​NULL​);
​ ​​ ​​ ​​ ​f​->​f_stacktop​ ​​= NULL​;​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​/*​ ​remains​ ​NULL​ ​unless​ ​yield​ ​suspends​ ​frame
*/
​ ​​ ​​ ​​ ​f​->​f_executing​ ​​=​​ ​​1;
...
```

**Observation  1**

**Points** ​ ​ **to** ​ ​ **the** ​ ​ **opcode** ​ ​ **to** ​ ​ **be** ​ ​ **executed** ​ ​ **for** ​ ​ **the** ​ ​ **current** ​ ​ **function.**

**Observation  2**

**The** ​ ​ **names** ​ ​ **of** ​ ​ **variables** ​ ​ **in** ​ ​ **the** ​ ​ **context** ​ ​ **used** ​ ​ **for** ​ ​ **storing** ​ ​ **and** ​ ​ **loading** ​ ​ **values** ​ ​ **of** ​ ​ **names**
```c
 ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​TARGET​(​STORE_NAME​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​name​ ​​=​​ ​GETITEM​(​names​,​​ ​oparg​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​v​ ​​=​​ ​POP​();
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​ns​ ​​=​​ ​f​->​f_locals;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​int​​ ​err;
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​....
```
**Observation  3**

**The** ​ ​ **constants** ​ ​ **in** ​ ​ **the** ​ ​ **current** ​ ​ **namespace** ​ ​ **used** ​ ​ **for** ​ ​ **loading** ​ ​ **constants** ​ ​ **computed** ​ ​ **in** ​ ​ **the** ​ ​ **current function** ​ ​ **scope.**

```c
PREDICTED​(​LOAD_CONST​);
   ​​ ​​ ​​ ​​ ​TARGET​(​LOAD_CONST​)​​ ​{
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​PyObject​​ ​​*​value​ ​​=​​ ​GETITEM​(​consts​,​​ ​oparg​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_INCREF​(​value​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​PUSH​(​value​);
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​FAST_DISPATCH​();

     ​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​}
```
**Observation  4**

**The** ​ ​ **local** ​ ​ **variables** ​ ​ **for** ​ ​ **the** ​ ​ **current** ​ ​ **scope.**

```c
/*​ ​Local​ ​variable​ ​macros​ ​*/

#define​​ ​GETLOCAL​(​i​)​​ ​​ ​​ ​​ ​​ ​​(​fastlocals​[​i​])

/*​ ​The​ ​SETLOCAL()​ ​macro​ ​must​ ​not​ ​DECREF​ ​the​ ​local​ ​variable​ ​in-place​ ​and
​ ​​ ​​ ​​then​​ ​store​ ​the​ ​​new​​ ​value​;​​ ​it​ ​must​ ​copy​ ​the​ ​old​ ​value​ ​to​ ​a​ ​temporary
​ ​​ ​​ ​value​,​​ ​​then​​ ​store​ ​the​ ​​new​​ ​value​,​​ ​​and​​ ​​then​​ ​DECREF​ ​the​ ​temporary​ ​value.
​ ​​ ​​ ​​This​​ ​​is​​ ​because​ ​it​ ​​is​​ ​possible​ ​that​ ​during​ ​the​ ​DECREF​ ​the​ ​frame​ ​​is
​ ​​ ​​ ​accessed​ ​​by​​ ​other​ ​code​ ​​(​e​.​g​.​​ ​a​ ​__del__​ ​method​ ​​or​​ ​gc​.​collect​())​​ ​​and​​ ​the
​ ​​ ​​ ​variable​ ​would​ ​be​ ​pointing​ ​to​ ​already​-​freed​ ​memory​.​​ ​​*/
#define​​ ​SETLOCAL​(​i​,​​ ​value​)​​ ​​ ​​ ​​ ​​ ​​ ​​do​​ ​​{​​ ​​PyObject​​ ​​*​tmp​ ​​=​​ ​GETLOCAL​(​i​);​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​GETLOCAL​(​i​)​​ ​​=​​ ​value​;​​ ​\
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​Py_XDECREF​(​tmp​);​​ ​​}​​ ​​while​​ ​​(​0)
```
**Observation  5**

**The** ​ ​ **pointer** ​ ​ **to** ​ ​ **the** ​ ​ **first** ​ ​ **instruction** ​ ​ **of** ​ ​ **the** ​ ​ **code** ​ ​ **for** ​ ​ **the** ​ ​ **function** ​ ​ **used** ​ ​ **for** ​ ​ **computing** ​ ​ **jump offsets**

```c
#define​​ ​JUMPTO​(​x​)​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​(​next_instr​ ​​=​​ ​first_instr​ ​​+​​ ​​(​x​)​​ ​​/
sizeof​(​_Py_CODEUNIT​))
```
**Observation  6**

**The** ​ ​ **stack** ​ ​ **pointer** ​ ​ **for** ​ ​ **all** ​ ​ **stack** ​ ​ **operations** ​ ​ **we** ​ ​ **shall** ​ ​ **cover** ​ ​ **all** ​ ​ **the** ​ ​ **operations** ​ ​ **on** ​ ​ **the** ​ ​ **stack** ​ ​ **in** ​ ​ **the coming** ​ ​ **chapters.**

```c
#define​​ ​EMPTY​()​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​(​STACK_LEVEL​()​​ ​​==​​ ​​0)
#define​​ ​TOP​()​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​(​stack_pointer​[-​ 1 ​])
#define​​ ​SECOND​()​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​(​stack_pointer​[-​ 2 ​])
#define​​ ​THIRD​()​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​​(​stack_pointer​[-​ 3 ​])
```
### Topic ​ ​ How ​ ​ are ​ ​ the ​ ​ stack ​ ​ frames ​ ​ arranged ​ ​ during ​ ​ interpretation ​ ​?

**The** ​ ​ **stack** ​ ​ **frames** ​ ​ **in** ​ ​ **python** ​ ​ **are** ​ ​ **arranged** ​ ​ **in** ​ ​ **python** ​ ​ **as** ​ ​ **a** ​ ​ **stack** ​ ​ **using** ​ ​ **reverse** ​ ​ **pointers** ​ ​ **to** ​ ​ **the parent** ​ ​ **function** ​ ​ **frame.**


**frameobject.c** ​ ​ **line** ​ ​ **no** ​ ​ **608**

```c
PyFrameObject​​ ​*
PyFrame_New​(​PyThreadState​​ ​​*​tstate​,​​ ​​PyCodeObject​​ ​​*​code​,​​ ​​PyObject​​ ​​*​globals, ​​PyObject​​ ​​*​locals)
{
​ ​​ ​​ ​​ ​​PyFrameObject​​ ​​*​back​ ​​=​​ ​tstate​->​frame​;
​ ​​ ​​ ​​ ​​....
​ ​​ ​​ ​​ ​​ ​​ ​​ ​f​->​f_back​ ​​=​​ ​back​;​​ ​​//​ ​ 1
​ ​​ ​​ ​​ ​​....
}
```
**1. The** ​ ​ **f_back** ​ ​ **pointer** ​ ​ **of** ​ ​ **the** ​ ​ **current** ​ ​ **frame** ​ ​ **ready** ​ ​ **for** ​ ​ **execution** ​ ​ **points** ​ ​ **to** ​ ​ **the** ​ ​ **function**  **frame** ​ ​ **that** ​ ​ **called** ​ ​ **it.**

**ceval.c** ​ ​ **line** ​ ​ **no** ​ ​ **3684**

```c
....
exit_eval_frame:
​ ​​ ​​ ​​ ​​if​​ ​​(​PyDTrace_FUNCTION_RETURN_ENABLED​())
​ ​​ ​​ ​​ ​​ ​​ ​​ ​​ ​dtrace_function_return​(​f​);
​ ​​ ​​ ​​ ​​Py_LeaveRecursiveCall​();
​ ​​ ​​ ​​ ​f​->​f_executing​ ​​=​​ ​​0;
​ ​​ ​​ ​​ ​tstate​->​frame​ ​​=​​ ​f​->​f_back;
....
```
**After** ​ ​ **the** ​ ​ **execution** ​ ​ **of** ​ ​ **the** ​ ​ **current** ​ ​ **function,** ​ ​ **the** ​ ​ **interpreter** ​ ​ **sets** ​ ​ **the** ​ ​ **current** ​ ​ **frame** ​ ​ **to** ​ ​ **be frame** ​ ​ **of** ​ ​ **the** ​ ​ **function** ​ ​ **that** ​ ​ **called** ​ ​ **the** ​ ​ **current** ​ ​ **function.**

**That’s ​ ​ all ​ ​ about ​ ​ interpreters. ​ ​ Feel ​ ​ free ​ ​ to ​ ​ explore ​ ​ further ​ ​ :).**
