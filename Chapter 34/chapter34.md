# Chapter​ ​ 34
# Python​ ​ sockets
### Listing​ ​ 1.​ ​ The​ ​ python​ ​ socket​ ​ wrapper​ ​ object​ ​ socketmodule.h​ ​ line​ ​ no​ ​ 168.
typedef​​ ​ struct​​ ​ {
​ ​ ​ ​ PyObject_HEAD
​ ​ ​ ​ SOCKET_T​ ​ sock_fd​ ; ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ Socket​ ​ file​ ​ descriptor​ ​ */
​ ​ ​ ​ int​​ ​ sock_family​ ; ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ Address​ ​ family,​ ​ e.g.,​ ​ AF_INET​ ​ */
​ ​ ​ ​ int​​ ​ sock_type​ ; ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ Socket​ ​ type,​ ​ e.g.,​ ​ SOCK_STREAM​ ​ */
​ ​ ​ ​ int​​ ​ sock_proto​ ; ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ Protocol​ ​ type,​ ​ usually​ ​ 0 ​ ​ */
​ ​ ​ ​ PyObject​​ ​ *(*​ errorhandler​ )(​ void​ );​​ ​ /*​ ​ Error​ ​ handler;​ ​ checks
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ errno​ , ​ ​ returns​ ​ NULL​ ​ and
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sets​ ​ a ​ ​ Python​​ ​ exception​ ​ */
​ ​ ​ ​ _PyTime_t​​ ​ sock_timeout​ ; ​ ​ ​ ​ ​ ​ /*​ ​ Operation​ ​ timeout​ ​ in​ ​ seconds;
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ 0.0​​ ​ means​ ​ non​ - ​ blocking​ ​ */
}​​ ​ PySocketSockObject;

### Listing​ ​ .2​ ​ The​ ​ type​ ​ object​ ​ for​ ​ sockets
static​​ ​ PyTypeObject​​ ​ sock_type​ ​ = ​ ​ {
​ ​ ​ ​ PyVarObject_HEAD_INIT​ ( ​ 0 ​ , ​ ​ 0 ​ ) ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ Must fill​ ​ in​ ​ type​ ​ value​ ​ later​ ​ */
​ ​ ​ ​ "_socket.socket"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​  /*​ ​ tp_name​ ​ */
​ ​ ​ ​ sizeof​ ( ​ PySocketSockObject​ ),​​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ tp_basicsize​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_itemsize​ ​ */
​ ​ ​ ​ ( ​ destructor​ ) ​ sock_dealloc​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_dealloc​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_print​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_getattr​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_setattr​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_reserved​ ​ */
​ ​ ​ ​ ( ​ reprfunc​ ) ​ sock_repr​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_repr​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_as_number​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_as_sequence​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_as_mapping​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_hash​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_call​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_str​ ​ */
​ ​ ​ ​ PyObject_GenericGetAttr​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_getattro​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_setattro​ ​ */
​ ​ ​ ​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ ​ tp_as_buffer​ ​ */
​ ​ ​ ​ Py_TPFLAGS_DEFAULT​​ ​ | ​ ​ Py_TPFLAGS_BASETYPE/*​ t
​ ​ ​ ​ ​ ​ ​ ​ | ​ ​ Py_TPFLAGS_HAVE_FINALIZE​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​p_flags​ ​ */
​ ​ ​ ​ sock_doc​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*​ tp_doc​ ​ */
0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_traverse​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_clear​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_richcompare​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_weaklistoffset​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_iter​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_iternext​ ​ */
​ sock_methods​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_methods​ ​ */
​ sock_memberlist​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_members​ ​ */
​ sock_getsetlist​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_getset​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_base​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_dict​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_descr_get​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_descr_set​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_dictoffset​ ​ */
​ sock_initobj​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_init​ ​ */
​ PyType_GenericAlloc​ , ​ ​ ​ ​ ​ ​/*tp_alloc​ ​ */
​ sock_new​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_new​ ​ */
​ PyObject_Del​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_free​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_is_gc​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_bases​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_mro​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_cache​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_subclasses​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_weaklist​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_del​ ​ */
​ 0 ​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​/*tp_version_tag​ ​ */
​ ( ​ destructor​ ) ​ sock_finalize​ , ​/*tp_finalize​ ​ */
};

### Listing​ ​ .3​ ​ The​ ​ methods​ ​ and​ ​ member​ ​ definitions​ ​ of​ ​ socket​ ​ objects
static​​ ​ PyMethodDefsock_methods​ []​​ ​ = ​ ​ {
​ ​ ​ ​ { ​ "_accept"​ , ​ ​ ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_accept​ , ​ ​ METH_NOARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​​ ​ ​ accept_doc​ },
​ ​ ​ ​ { ​ "bind"​ , ​ ​ ​ ​ ​ ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_bind​ , ​ ​ METH_O,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​​ ​ ​ bind_doc​ },
​ ​ ​ ​ { ​ "close"​ , ​ ​ ​ ​ ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_close​ , ​ ​ METH_NOARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​​ ​ ​ close_doc​ },
​ ​ ​ ​ { ​ "connect"​ , ​ ​ ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_connect​ , ​ ​ METH_O,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​​ ​ ​ connect_doc​ },
​ ​ ​ ​ { ​ "connect_ex"​ , ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_connect_ex​ , ​ ​ METH_O,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​​ ​ ​ connect_ex_doc​ },
​ ​ ​ ​ { ​ "detach"​ , ​ ​ ​ ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_detach​ , ​ ​ METH_NOARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​​ ​ ​ detach_doc​ },
​ ​ ​ ​ { ​ "fileno"​ , ​ ​ ​ ​​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_fileno​ , ​ ​ METH_NOARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ fileno_doc​ },
#ifdef​​ ​ HAVE_GETPEERNAME
​ ​ ​ ​ { ​ "getpeername"​ , ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_getpeername,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ METH_NOARGS​ , ​ ​ getpeername_doc​ },
#endif
​ ​ ​ ​ { ​ "getsockname"​ , ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_getsockname,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ METH_NOARGS​ , ​ ​ getsockname_doc​ },
​ ​ ​ ​ { ​ "getsockopt"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_getsockopt​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ getsockopt_doc​ },
#if​ ​ defined(MS_WINDOWS)​ ​ &&​ ​ defined(SIO_RCVALL)
​ ​ ​ ​ { ​ "ioctl"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_ioctl​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sock_ioctl_doc​ },
#endif
#if​ ​ defined(MS_WINDOWS)
​ ​ ​ ​ { ​ "share"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_share​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sock_share_doc​ },
#endif
​ ​ ​ ​ { ​ "listen"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_listen​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ listen_doc​ },
​ ​ ​ ​ { ​ "recv"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_recv​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ recv_doc​ },
​ ​ ​ ​ { ​ "recv_into"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_recv_into​ , ​ ​ METH_VARARGS​ ​ |
METH_KEYWORDS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ recv_into_doc​ },
​ ​ ​ ​ { ​ "recvfrom"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_recvfrom​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ recvfrom_doc​ },
​ ​ ​ ​ { ​ "recvfrom_into"​ , ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_recvfrom_into​ , ​ ​ METH_VARARGS​ ​ |
METH_KEYWORDS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ recvfrom_into_doc​ },
​ ​ ​ ​ { ​ "send"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_send​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ send_doc​ },
​ ​ ​ ​ { ​ "sendall"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_sendall​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sendall_doc​ },
​ ​ ​ ​ { ​ "sendto"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_sendto​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sendto_doc​ },
​ ​ ​ ​ { ​ "setblocking"​ , ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_setblocking​ , ​ ​ METH_O,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ setblocking_doc​ },
​ ​ ​ ​ { ​ "settimeout"​ , ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_settimeout​ , ​ ​ METH_O,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ settimeout_doc​ },
​ ​ ​ ​ { ​ "gettimeout"​ , ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_gettimeout​ , ​ ​ METH_NOARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ gettimeout_doc​ },
​ ​ ​ ​ { ​ "setsockopt"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_setsockopt​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ setsockopt_doc​ },
​ ​ ​ ​ { ​ "shutdown"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_shutdown​ , ​ ​ METH_O,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ shutdown_doc​ },
#ifdef​​ ​ CMSG_LEN
​ ​ ​ ​ { ​ "recvmsg"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_recvmsg​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ recvmsg_doc​ },
​ ​ ​ ​ { ​ "recvmsg_into"​ , ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_recvmsg_into​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ recvmsg_into_doc​ ,},
​ ​ ​ ​ { ​ "sendmsg"​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_sendmsg​ , ​ ​ METH_VARARGS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sendmsg_doc​ },
#endif
#ifdef​​ ​ HAVE_SOCKADDR_ALG
​ ​ ​ ​ { ​ "sendmsg_afalg"​ , ​ ​ ​ ​ ​ ​ ( ​ PyCFunction​ ) ​ sock_sendmsg_afalg​ , ​ ​ METH_VARARGS​ ​ |
METH_KEYWORDS,
​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ sendmsg_afalg_doc​ },
#endif
​ ​ ​ ​ { ​ NULL​ , ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ NULL​ } ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ ​ /*​ ​ sentinel​ ​ */
};
/*​ ​ SockObject​ ​ members​ ​ */
static​​ ​ PyMemberDef​​ ​ sock_memberlist​ []​​ ​ = ​ ​ {
​ ​ ​ ​ ​ ​ ​ { ​ "family"​ , ​ ​ T_INT​ , ​ ​ offsetof​ ( ​ PySocketSockObject​ , ​ ​ sock_family​ ),
READONLY​ , ​ ​ "the​ ​ socket​ ​ family"​ },
​ ​ ​ ​ ​ ​ ​ { ​ "type"​ , ​ ​ T_INT​ , ​ ​ offsetof​ ( ​ PySocketSockObject​ , ​ ​ sock_type​ ),​​ ​ READONLY​ ,
"the​ ​ socket​ ​ type"​ },
​ ​ ​ ​ ​ ​ ​ { ​ "proto"​ , ​ ​ T_INT​ , ​ ​ offsetof​ ( ​ PySocketSockObject​ , ​ ​ sock_proto​ ),​​ ​ READONLY​ ,
"the​ ​ socket​ ​ protocol"​ },
​ ​ ​ ​ ​ ​ ​ { ​ 0 ​ },
};

We​ ​ shall​ ​ understand​ ​ each​ ​ of​ ​ these​ ​ members​ ​ and​ ​ methods​ ​ in​ ​ the​ ​ upcoming​ ​ sections.
The​ ​ interface​ ​ to​ ​ the​ ​ python​ ​ socket​ ​ object​ ​ is​ ​ provided​ ​ by​ ​ the​ ​ python​ ​ standard​ ​ library
Modules/socket.py

class​​ ​ socket​ ( ​ _socket​ . ​ socket​ ):​​ ​ ​ //​ ​ 1
​ ​ ​ ​ """A​ ​ subclass​ ​ of​ ​ _socket.socket​ ​ adding​ ​ the​ ​ makefile()​ ​ method."""
​ ​ ​ ​ __slots__​ ​ = ​ ​ [ ​ "__weakref__"​ , ​ ​ "_io_refs"​ , ​ ​ "_closed"]
​ ​ ​ ​ def​​ ​ __init__​ ( ​ self​ , ​ ​ family​ = ​ AF_INET​ , ​ ​ type​ = ​ SOCK_STREAM​ , ​ ​ proto​ = ​ 0 ​ ,
fileno​ = ​ None​ ):
​ ​ ​ ​ ​ ​ ​ ​ # ​ ​ For​ ​ user​ ​ code​ ​ address​ ​ family​ ​ and​ ​ type​ ​ values​ ​ are​ ​ IntEnum​ ​ members,
but
​ ​ ​ ​ ​ ​ ​ ​ # ​ ​ for​ ​ the​ ​ underlying​ ​ _socket.socket​ ​ they're​ ​ just​ ​ integers.​ ​ The
​ ​ ​ ​ ​ ​ ​ ​ # ​ ​ constructor​ ​ of​ ​ _socket.socket​ ​ converts​ ​ the​ ​ given​ ​ argument​ ​ to​ ​ an
​ ​ ​ ​ ​ ​ ​ ​ # ​ ​ integer​ ​ automatically.
​ ​ ​ ​ ​ ​ ​ ​ _socket​ . ​ socket​ . ​ __init__​ ( ​ self​ , ​ ​ family​ , ​ ​ type​ , ​ ​ proto​ , ​ ​ fileno​ ) ​ ​ //​ ​ 2
​ ​ ​ ​ ​ ​ ​ self​ . ​ _io_refs​ ​ = ​ ​ 0
​ ​ ​ ​ ​ ​ ​ ​ self​ . ​ _closed​ ​ = ​ ​ False
​ ​ ​ ​ def​​ _
_enter__​ ( ​ self​ ):
​ ​ ​ ​ ​ ​ ​ ​ return​​ ​ self
* The​​ ​ socket​ ​ class​​ ​ extends​​ ​ the​ b
asic​ ​ python​ ​ type​ ​ ' ​ _socket​ . ​ socket​ '
defined​​ ​ in​​ ​ the​ ​ listing​ ​ .2
* It​​ ​ initializes​ ​ a ​ ​ call​ ​ to​ ​ the​ i
nit​ ​ function​​ ​ of​ ​ the​ ​ super​​ ​ class
####Debugging​ ​ session
The​ ​ socket​ ​ is​ ​ initialized​ ​ by​ ​ the​ ​ function​ ​ sock_new​ ​ in​ ​ socketmodule.c​ ​ on​ ​ line​ ​ no​ ​ 4625

static​​ ​ PyObject​​ ​ *
sock_new​ ( ​ PyTypeObject​​ ​ * ​ type​ , ​ ​ PyObject​​ ​ * ​ args​ , ​ ​ PyObject​​ ​ * ​ kwds)
{
​ ​ ​ ​ PyObject​​ ​ * ​ new;
​ ​ ​ ​ new​​ ​ = ​ ​ type​ ->​ tp_alloc​ ( ​ type​ , ​ ​ 0 ​ );​​ ​ //​ ​ 1
​ ​ ​ ​ if​​ ​ ( ​ new​​ ​ !=​​ ​ NULL​ ) ​ ​ { ​ ​ //​ ​ 2
​ ​ ​ ​ ​ ​ ​ ​ ((​ PySocketSockObject​​ ​ *)​ new​ )->​ sock_fd​ ​ = ​ ​ INVALID_SOCKET;
​ ​ ​ ​ ​ ​ ​ ​ ((​ PySocketSockObject​​ ​ *)​ new​ )->​ sock_timeout​ ​ =
_PyTime_FromSeconds​ (-​ 1 ​ );
​ ​ ​ ​ ​ ​ ​ ​ ((​ PySocketSockObject​​ ​ *)​ new​ )->​ errorhandler​ ​ = ​ ​ & ​ set_error;
​ ​ ​ ​ }
​ ​ ​ ​ return​​ ​ new;
}
The​​ ​ new​​ ​ function​​ ​ of​ ​ socket​ ​ object​​ ​ is​​ ​ defined​​ ​ as​​ ​ sock_new
1. tp_alloc​ ​ ->​​ ​ PyType_GenericAlloc
2. Initialization​​ ​ of​ ​ variables

static​​ ​ int
sock_initobj​ ( ​ PyObject​​ ​ * ​ self​ , ​ ​ PyObject​​ ​ * ​ args​ , ​ ​ PyObject​​ ​ * ​ kwds)
{
​ ​ ​ ​ PySocketSockObject​​ ​ * ​ s ​ ​ = ​ ​ ( ​ PySocketSockObject​​ ​ *)​ self;
​ ​ ​ ​ PyObject​​ ​ * ​ fdobj​ ​ = ​ ​ NULL;
​ ​ ​ ​ SOCKET_T​ ​ fd​ ​ = ​ ​ INVALID_SOCKET;
​ ​ ​ ​ int​​ ​ family​ ​ = ​ ​ AF_INET​ , ​ ​ type​ ​ = ​ ​ SOCK_STREAM​ , ​ ​ proto​ ​ = ​ ​ 0;
​ ​ ​ ​ static​​ ​ char​​ ​ * ​ keywords​ []​​ ​ = ​ ​ { ​ "family"​ , ​ ​ "type"​ , ​ ​ "proto"​ , ​ ​ "fileno"​ , ​ ​ 0 ​ };

#ifdef MS_WINDOWS
/* recreate a socket that was duplicated */
if (PyBytes_Check(fdobj)) {
WSAPROTOCOL_INFO info;
if (PyBytes_GET_SIZE(fdobj) != sizeof(info)) {
PyErr_Format(PyExc_ValueError,
"socket descriptor string has wrong size, "
"should be %zu bytes.", sizeof(info));
return -1;
}
memcpy(&info, PyBytes_AS_STRING(fdobj), sizeof(info));
Py_BEGIN_ALLOW_THREADS
fd = WSASocket(FROM_PROTOCOL_INFO, FROM_PROTOCOL_INFO,
FROM_PROTOCOL_INFO, &info, 0, WSA_FLAG_OVERLAPPED);
Py_END_ALLOW_THREADS
if (fd == INVALID_SOCKET) {
set_error();
return -1;
}
family = info.iAddressFamily;
type = info.iSocketType;
proto = info.iProtocol;
}
else
#endif
{
fd = PyLong_AsSocket_t(fdobj);
if (fd == (SOCKET_T)(-1) && PyErr_Occurred())
return -1;
if (fd == INVALID_SOCKET) {
PyErr_SetString(PyExc_ValueError,
"can't use invalid socket value");
return -1;
}
}
}
else {
#ifdef MS_WINDOWS
/* Windows implementation */
#ifndef WSA_FLAG_NO_HANDLE_INHERIT
#define WSA_FLAG_NO_HANDLE_INHERIT 0x80
#endif
Py_BEGIN_ALLOW_THREADS
if (support_wsa_no_inherit) {
fd = WSASocket(family, type, proto,
NULL, 0,
WSA_FLAG_OVERLAPPED |
WSA_FLAG_NO_HANDLE_INHERIT);
if (fd == INVALID_SOCKET) {
/* Windows 7 or Windows 2008 R2 without SP1 or the hotfix
*/
support_wsa_no_inherit = 0;
fd = socket(family, type, proto);
}
}
else {
fd = socket(family, type, proto); // 1
}
Py_END_ALLOW_THREADS
if (fd == INVALID_SOCKET) {
set_error();
return -1;
}
if (!support_wsa_no_inherit) {
if (!SetHandleInformation((HANDLE)fd, HANDLE_FLAG_INHERIT, 0))
{
closesocket(fd);
PyErr_SetFromWindowsErr(0);
return -1;
}
}
#else
/* UNIX */
Py_BEGIN_ALLOW_THREADS
#ifdef SOCK_CLOEXEC
if (sock_cloexec_works != 0) {
fd = socket(family, type | SOCK_CLOEXEC, proto);
if (sock_cloexec_works == -1) {
if (fd >= 0) {
sock_cloexec_works = 1;
}
else if (errno == EINVAL) {
/* Linux older than 2.6.27 does not support
SOCK_CLOEXEC */
sock_cloexec_works = 0;
fd = socket(family, type, proto);
}
}
}
else
#endif
{
fd = socket(family, type, proto);
}
Py_END_ALLOW_THREADS
if (fd == INVALID_SOCKET) {
set_error();
return -1;
}
if (_Py_set_inheritable(fd, 0, atomic_flag_works) < 0) {
SOCKETCLOSE(fd);
return -1;
}
#endif
}
if (init_sockobject(s, fd, family, type, proto) == -1) { // 2
SOCKETCLOSE(fd);
return -1;
}
return 0;
}
1. Create the socket object
2. add the wrapper information to the structure
static int
init_sockobject(PySocketSockObject *s,
SOCKET_T fd, int family, int type, int proto)
{
s->sock_fd = fd;s->sock_family = family;
s->sock_type = type;
s->sock_proto = proto;
s->errorhandler = &set_error;
#ifdef SOCK_NONBLOCK
if (type & SOCK_NONBLOCK)
s->sock_timeout = 0;
else
#endif
{
s->sock_timeout = defaulttimeout;
if (defaulttimeout >= 0) {
if (internal_setblocking(s, 0) == -1) {
return -1;
}
}
}
return 0;
}

s.connect(("www.python.org", 80))
static int
internal_connect(PySocketSockObject *s, struct sockaddr *addr, int addrlen,
int raise)
{
int res, err, wait_connect;
Py_BEGIN_ALLOW_THREADS
res = connect(s->sock_fd, addr, addrlen); // 1
Py_END_ALLOW_THREADS
if (!res) {
/* connect() succeeded, the socket is connected */
return 0;
}
/* connect() failed */
/* save error, PyErr_CheckSignals() can replace it */
err = GET_SOCK_ERROR;
if (CHECK_ERRNO(EINTR)) {
if (PyErr_CheckSignals())
return -1;
/* Issue #23618: when connect() fails with EINTR, the connection is
running asynchronously.
If the socket is blocking or has a timeout, wait until the
connection completes, fails or timed out using select(), and
then
get the connection status using getsockopt(SO_ERROR).
If the socket is non-blocking, raise InterruptedError. The
caller is
responsible to wait until the connection completes, fails or
timed
out (it's the case in asyncio for example). */
wait_connect = (s->sock_timeout != 0 && IS_SELECTABLE(s));
}
else {
wait_connect = (s->sock_timeout > 0 && err == SOCK_INPROGRESS_ERR
&& IS_SELECTABLE(s));
}
if (!wait_connect) {
if (raise) {
/* restore error, maybe replaced by PyErr_CheckSignals() */
SET_SOCK_ERROR(err);
s->errorhandler();
return -1;
}
else
return err;
}
if (raise) {
/* socket.connect() raises an exception on error */
if (sock_call_ex(s, 1, sock_connect_impl, NULL,
1, NULL, s->sock_timeout) < 0) // 2
return -1;
}
else {
/* socket.connect_ex() returns the error code on error */
if (sock_call_ex(s, 1, sock_connect_impl, NULL,
1, &err, s->sock_timeout) < 0)
return err;
}
return 0;
}
1. Connect the socket.
2. Raise the exception in case of any errors

Let us understand a bit about server sockets

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
serversocket.bind((socket.gethostname(), 80))
static PyObject *
socket_gethostname(PyObject *self, PyObject *unused)
{
#ifdef MS_WINDOWS
/* Don't use winsock's gethostname, as this returns the ANSI
version of the hostname, whereas we need a Unicode string.
Otherwise, gethostname apparently also returns the DNS name. */
wchar_t buf[MAX_COMPUTERNAME_LENGTH + 1];
DWORD size = Py_ARRAY_LENGTH(buf);
wchar_t *name;
PyObject *result;
if (GetComputerNameExW(ComputerNamePhysicalDnsHostname, buf, &size))
return PyUnicode_FromWideChar(buf, size);
if (GetLastError() != ERROR_MORE_DATA)
return PyErr_SetFromWindowsErr(0);
if (size == 0)

return PyUnicode_New(0, 0);
/* MSDN says ERROR_MORE_DATA may occur because DNS allows longer
names */
name = PyMem_New(wchar_t, size);
if (!name) {
PyErr_NoMemory();
return NULL;
}
if (!GetComputerNameExW(ComputerNamePhysicalDnsHostname,
name,
&size))
{
PyMem_Free(name);
return PyErr_SetFromWindowsErr(0);
}
result = PyUnicode_FromWideChar(name, size);
PyMem_Free(name);
return result;
#else
char buf[1024];
int res;
Py_BEGIN_ALLOW_THREADS
res = gethostname(buf, (int) sizeof buf - 1); // 1
Py_END_ALLOW_THREADS
if (res < 0)
return set_error();
buf[sizeof buf - 1] = '\0';
return PyUnicode_DecodeFSDefault(buf);
#endif
}
static PyObject *
sock_bind(PySocketSockObject *s, PyObject *addro)
{
sock_addr_t addrbuf;
int addrlen;
int res;
if (!getsockaddrarg(s, addro, SAS2SA(&addrbuf), &addrlen))
return NULL;
Py_BEGIN_ALLOW_THREADS
res = bind(s->sock_fd, SAS2SA(&addrbuf), addrlen); // 2
Py_END_ALLOW_THREADS
if (res < 0)
return s->errorhandler();

Py_INCREF(Py_None);
return Py_None;
}
1. Get the hostname of the current machine
2. Bind to the address and port

serversocket.listen(5)
static PyObject *
sock_listen(PySocketSockObject *s, PyObject *args)
{
/* We try to choose a default backlog high enough to avoid connection
drops
* for common workloads, yet not too high to limit resource usage. */
int backlog = Py_MIN(SOMAXCONN, 128);
int res;
if (!PyArg_ParseTuple(args, "|i:listen", &backlog))
return NULL;
Py_BEGIN_ALLOW_THREADS
/* To avoid problems on systems that don't allow a negative backlog
* (which doesn't make sense anyway) we force a minimum value of 0. */
if (backlog < 0)
backlog = 0;
res = listen(s->sock_fd, backlog); // 1
Py_END_ALLOW_THREADS
if (res < 0)
return s->errorhandler();
Py_INCREF(Py_None);
return Py_None;
}
1. Start listening on the socket.

[![N|Solid](chapter34/img/1.png)]

(​ clientsocket​ , ​ ​ address​ ) ​ ​ = ​ ​ serversocket​ . ​ accept​ ()
static​​ ​ PyObject​​ ​ *
sock_accept​ ( ​ PySocketSockObject​​ ​ * ​ s)
{
​ ​ ​ ​ sock_addr_t​​ ​ addrbuf;
​ ​ ​ ​ SOCKET_T​ ​ newfd;
​ ​ ​ ​ socklen_t​​ ​ addrlen;
​ ​ ​ ​ PyObject​​ ​ * ​ sock​ ​ = ​ ​ NULL;
​ ​ ​ ​ PyObject​​ ​ * ​ addr​ ​ = ​ ​ NULL;
​ ​ ​ ​ PyObject​​ ​ * ​ res​ ​ = ​ ​ NULL;
​ ​ ​ ​ struct​​ ​ sock_accept​ ​ ctx;
​ ​ ​ ​ if​​ ​ (!​ getsockaddrlen​ ( ​ s ​ , ​ ​ & ​ addrlen​ ))
​ ​ ​ ​ ​ ​ ​ ​ return​​ ​ NULL;
​ ​ ​ m
emset​ (&​ addrbuf​ , ​ ​ 0 ​ , ​ ​ addrlen​ );
​ ​ ​ ​ if​​ ​ (!​ IS_SELECTABLE​ ( ​ s ​ ))
​ ​ ​ ​ ​ ​ ​ ​ return​​ ​ select_error​ ();
​
​
​
​
​ ctx​ . ​ addrlen​ ​ = ​ ​ & ​ addrlen;
​ ctx​ . ​ addrbuf​ ​ = ​ ​ & ​ addrbuf;
​ if​​ ​ ( ​ sock_call​ ( ​ s ​ , ​ ​ 0 ​ , ​ ​ sock_accept_impl​ , ​ ​ & ​ ctx​ ) ​ ​ < ​ ​ 0 ​ ) ​ ​ //​ ​ 1
​ ​ ​ ​ ​ return​​ ​ NULL;
newfd = ctx.result;
#ifdef MS_WINDOWS
if (!SetHandleInformation((HANDLE)newfd, HANDLE_FLAG_INHERIT, 0)) {
PyErr_SetFromWindowsErr(0);
SOCKETCLOSE(newfd);
goto finally;
}
#else
#if defined(HAVE_ACCEPT4) && defined(SOCK_CLOEXEC)
if (!accept4_works)
#endif
{
if (_Py_set_inheritable(newfd, 0, NULL) < 0) {
SOCKETCLOSE(newfd);
goto finally;
}
}
#endif
sock = PyLong_FromSocket_t(newfd);
if (sock == NULL) {
SOCKETCLOSE(newfd);
goto finally;
}
addr = makesockaddr(s->sock_fd, SAS2SA(&addrbuf),
addrlen, s->sock_proto);
if (addr == NULL)
goto finally;
res = PyTuple_Pack(2, sock, addr); // 2
finally:
Py_XDECREF(sock);
Py_XDECREF(addr);
return res;
}

Observation 1

static int
sock_call(PySocketSockObject *s,
int writing,
int (*func) (PySocketSockObject *s, void *data),
void *data)
{
return sock_call_ex(s, writing, func, data, 0, NULL, s->sock_timeout);
}
static int
sock_call_ex(PySocketSockObject *s,
int writing,
int (*sock_func) (PySocketSockObject *s, void *data),
void *data,
int connect,
int *err,
_PyTime_t timeout)
{
int has_timeout = (timeout > 0);
_PyTime_t deadline = 0;
int deadline_initialized = 0;
int res;
#ifdef WITH_THREAD
/* sock_call() must be called with the GIL held. */
assert(PyGILState_Check());
#endif
/* outer loop to retry select() when select() is interrupted by a
signal
or to retry select()+sock_func() on false positive (see above) */
while (1) {
/* For connect(), poll even for blocking socket. The connection
runs asynchronously. */
if (has_timeout || connect) {
if (has_timeout) {
_PyTime_t interval;
if (deadline_initialized) {
/* recompute the timeout */
interval = deadline - _PyTime_GetMonotonicClock();
}
else {
deadline_initialized = 1;
deadline = _PyTime_GetMonotonicClock() + timeout;
interval = timeout;
}
if (interval >= 0)
res = internal_select(s, writing, interval, connect);// 1
else
res = 1;
}
else {
res = internal_select(s, writing, timeout, connect);
}
if (res == -1) {
if (err)
*err = GET_SOCK_ERROR;
if (CHECK_ERRNO(EINTR)) {
/* select() was interrupted by a signal */
if (PyErr_CheckSignals()) {
if (err)
*err = -1;
return -1;
}
/* retry select() */
continue;
}
/* select() failed */
s->errorhandler();
return -1;
}
if (res == 1) {
if (err)
*err = SOCK_TIMEOUT_ERR;
else
PyErr_SetString(socket_timeout, "timed out");
return -1;
}
/* the socket is ready */
}
/* inner loop to retry sock_func() when sock_func() is interrupted
by a signal */
while (1) {
Py_BEGIN_ALLOW_THREADS
res = sock_func(s, data); // 2
Py_END_ALLOW_THREADS
if (res) {
/* sock_func() succeeded */
if (err)
*err = 0;
return 0;
}
if (err)
*err = GET_SOCK_ERROR;
if (!CHECK_ERRNO(EINTR))
break;
/* sock_func() was interrupted by a signal */
if (PyErr_CheckSignals()) {
if (err)
*err = -1;
return -1;
}
/* retry sock_func() */
}
if (s->sock_timeout > 0
&& (CHECK_ERRNO(EWOULDBLOCK) || CHECK_ERRNO(EAGAIN))) {
/* False positive: sock_func() failed with EWOULDBLOCK or
EAGAIN.
For example, select() could indicate a socket is ready for
reading, but the data then discarded by the OS because of a
wrong checksum.
Loop on select() to recheck for socket readyness. */
continue;
}
/* sock_func() failed */
if (!err)
s->errorhandler();
/* else: err was already set before */
return -1;
}
}

1. Call the select function to poll upon the events for the socket
2. Call the function passed which is socket_accept_impl

struct pollfd
{
int fd; /* File descriptor to poll. */
short int events; /* Types of events poller cares about. */
short int revents; /* Types of events that actually occurred.
*/
};
static int
internal_select(PySocketSockObject *s, int writing, _PyTime_t interval,
int connect)
{
int n;
#ifdef HAVE_POLL
struct pollfd pollfd;
_PyTime_t ms;
#else
fd_set fds, efds;
struct timeval tv, *tvp;
#endif
#ifdef WITH_THREAD
/* must be called with the GIL held */
assert(PyGILState_Check());
#endif
/* Error condition is for output only */
assert(!(connect && !writing));
/* Guard against closed socket */
if (s->sock_fd == INVALID_SOCKET)
return 0;
/* Prefer poll, if available, since you can poll() any fd
* which can't be done with select(). */
#ifdef HAVE_POLL
pollfd.fd = s->sock_fd;
pollfd.events = writing ? POLLOUT : POLLIN;
if (connect) {
/* On Windows, the socket becomes writable on connection success,
but a connection failure is notified as an error. On POSIX, the
socket becomes writable on connection success or on connection
failure. */
pollfd.events |= POLLERR;
}
/* s->sock_timeout is in seconds, timeout in ms */
ms = _PyTime_AsMilliseconds(interval, _PyTime_ROUND_CEILING);
assert(ms <= INT_MAX);
Py_BEGIN_ALLOW_THREADS;
n = poll(&pollfd, 1, (int)ms); // 3
Py_END_ALLOW_THREADS;
#else
if (interval >= 0) {
_PyTime_AsTimeval_noraise(interval, &tv, _PyTime_ROUND_CEILING);
tvp = &tv;
}
else
tvp = NULL;
FD_ZERO(&fds);
FD_SET(s->sock_fd, &fds);
FD_ZERO(&efds);
if (connect) {
/* On Windows, the socket becomes writable on connection success,
but a connection failure is notified as an error. On POSIX, the
socket becomes writable on connection success or on connection
failure. */
FD_SET(s->sock_fd, &efds);
}
/* See if the socket is ready */
Py_BEGIN_ALLOW_THREADS;
if (writing)
n = select(Py_SAFE_DOWNCAST(s->sock_fd+1, SOCKET_T, int),
NULL, &fds, &efds, tvp);
else
n = select(Py_SAFE_DOWNCAST(s->sock_fd+1, SOCKET_T, int),
&fds, NULL, &efds, tvp);
Py_END_ALLOW_THREADS;
#endif
if (n < 0)
return -1;
if (n == 0)
return 1;
return 0;
}
1. poll on the socket to get the events
static int
sock_accept_impl(PySocketSockObject *s, void *data)
{
struct sock_accept *ctx = data;
struct sockaddr *addr = SAS2SA(ctx->addrbuf);
socklen_t *paddrlen = ctx->addrlen;
#ifdef HAVE_SOCKADDR_ALG
/* AF_ALG does not support accept() with addr and raises
* ECONNABORTED instead. */
if (s->sock_family == AF_ALG) {
addr = NULL;
paddrlen = NULL;
*ctx->addrlen = 0;
}
#endif
#if defined(HAVE_ACCEPT4) && defined(SOCK_CLOEXEC)
if (accept4_works != 0) {
ctx->result = accept4(s->sock_fd, addr, paddrlen,
SOCK_CLOEXEC);
if (ctx->result == INVALID_SOCKET && accept4_works == -1) {
/* On Linux older than 2.6.28, accept4() fails with ENOSYS */
accept4_works = (errno != ENOSYS);
}
}
if (accept4_works == 0)
ctx->result = accept(s->sock_fd, addr, paddrlen);
#else
ctx->result = accept(s->sock_fd, addr, paddrlen);
#endif
#ifdef MS_WINDOWS
return (ctx->result != INVALID_SOCKET);
#else
return (ctx->result >= 0);
#endif
}

Get peername of a socket

static PyObject *
sock_getpeername(PySocketSockObject *s)
{
sock_addr_t addrbuf;
int res;
socklen_t addrlen;
if (!getsockaddrlen(s, &addrlen))
return NULL;
memset(&addrbuf, 0, addrlen);
Py_BEGIN_ALLOW_THREADS
res = getpeername(s->sock_fd, SAS2SA(&addrbuf), &addrlen);
Py_END_ALLOW_THREADS
if (res < 0)
return s->errorhandler();
return makesockaddr(s->sock_fd, SAS2SA(&addrbuf), addrlen,
s->sock_proto);
}

Send data into a socket

static PyObject *
sock_send(PySocketSockObject *s, PyObject *args)
{
int flags = 0;
Py_buffer pbuf;
struct sock_send ctx;
if (!PyArg_ParseTuple(args, "y*|i:send", &pbuf, &flags))
return NULL;
if (!IS_SELECTABLE(s)) {
PyBuffer_Release(&pbuf);
return select_error();
}
ctx.buf = pbuf.buf;
ctx.len = pbuf.len;
ctx.flags = flags;
if (sock_call(s, 1, sock_send_impl, &ctx) < 0) { // 1
PyBuffer_Release(&pbuf);
return NULL;
}
PyBuffer_Release(&pbuf);
return PyLong_FromSsize_t(ctx.result);
}

1. Puts a request onto the event loop with the callback function
sock_send_impl.

Shutdown a socket

static PyObject *
sock_shutdown(PySocketSockObject *s, PyObject *arg)
{
int how;
int res;
how = _PyLong_AsInt(arg);
if (how == -1 && PyErr_Occurred())
return NULL;
Py_BEGIN_ALLOW_THREADS
res = shutdown(s->sock_fd, how); // 1
Py_END_ALLOW_THREADS
if (res < 0)
return s->errorhandler();
Py_INCREF(Py_None);
return Py_None;
}

1. Call the c standard library function to shutdown the socket.
