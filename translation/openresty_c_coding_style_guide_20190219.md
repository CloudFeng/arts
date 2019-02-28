<link href="markdown.css" rel="stylesheet"></link>

# OpenResty C 编码风格指南

OpenResty在C语言模块遵守NGINX的编码风格，比如OpenResty本身的NGINX插件模块或者OpenResty的Lua库中C部分。然而，即便是NGINX自身的C语言核心代码风格并没有与基础代码保持一致。所以，本文拟定一份指南性的文档以便消除各种歧义。

提交给OpenResty核心项目的补丁需要遵守此指南，否则不会通过评审进程，更不用说归并在一起。OpenRest和NGINX社区都鼓励开发者在使用C语言开发自身模块或库时遵守此指南。

## 命名规范

对于NIGINX相关的C代码，源码文件名字（包括`.c`和`.h`文件）、全局变量、全局函数、C语言中的结构/联合体/枚举的名字、编译单元作用域的静态变量和函数块，还有在头文件中公开定义的宏，以上这些都是全量命名。比如

```C
ngx_http_core_module.c
ngx_http_finalize_request
和 NGX_HTTP_MAIN_CONF
```

这是十分重要的，是因为在C语言中并不像C++中有显示的名字空间。使用全量性名字可以避免名字冲突，而且有助于调试。在Lua库中的C组件，对于所有在相关C编译单元中顶层C标签中我们同样使用前缀，比如`resty_blah_`（如果库的名称是`lua-resty-blah`）。

在C函数中声明局部变量使用的短名字。在NGINX核心组件中广泛使用短变量名字是：`cl`、`ev`、`p` 和 `q` 等等。这些变量名通常是生命周期比较短并且是有限的作用域。根据Huffman原则，在当前上下文中对于常用的地方我们使用短变量名可以避免行噪音。但短变量名需要遵守NGINX的风格。不要自我发明除非必要，但必须使用含义明确的名字。对于`p`和`q`,它们经常用在处理字符串处理上下文中的字符串指针变量名。

一旦需要C结构体和联合体，全写形式命名之，除非成员的名字很长。比如，在NGINX中 `struct ngx_http_request_s`，有很长的成员变量名字： `read_event_handler`、`upstream_states`、
和 `request_body_in_persistent_file` 等。

对于`typedef`指向的结构体的变量名以`_t`作为后缀，结构体名以`_s`作为后缀；对于`typedef`指向枚举的变量名以`_e`作为后缀。在函数作用域中定义的局部变量可以不必遵守此约定。下面是NGINX核心组件的代码：

```C
typedef struct ngx_connection_s      ngx_connection_t;
typedef struct {
    WSAOVERLAPPED    ovlp;
    ngx_event_t     *event;
    int              error;
} ngx_event_ovlp_t;
struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
};
typedef enum {
    ngx_pop3_start = 0,
    ngx_pop3_user,
    ...
    ngx_pop3_auth_external
} ngx_pop3_state_e;
```

## 缩进

NGINX中使用空格作为缩进，而不是使用制表符。通常我们使用4个空格，除了某些地方有对齐要求或者在特定的类中需要（我们会在后面详细讨论这些示例）。总之，合理的缩进你的代码。

## 每行限定80个字符

所有的源码行必须控制在80个字符内（尽管在NGINX中有些地方保持是78个，但建议定死80个字符）。不同场景下，对于缩进在连续行中有不同的缩进规则。我我们将在下面详细讨论之。

## 每行结尾无空白符

代码行尾不应有任何空格、制表符，甚至空行。很多编辑器会在用户设置下对空白字符自动高亮或者截断。合理地配置你的编辑器或者IDE。

## 函数声明

将头文件或者`.c`文件头部的C函数声明（不是定义！）尽量放在一行。下面是来自NGINX中的示例：

`ngx_int_t ngx_http_send_special(ngx_http_request_t *r, ngx_uint_t flags);`

若一行太长，超过了80个字符，可以使用4个空格符将其分隔多行。比如：

```C
ngx_int_t ngx_http_filter_finalize_request(ngx_http_request_t *r,
    ngx_module_t *m, ngx_int_t error);
```

若返回类型是指针类型，需要`*`之前有个空格而不是其后，如下所示：

`char *ngx_http_types_slot(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);`

注意函数定义与声明是不同的风格， 更多可以参考 [函数定义](#函数定义)部分 。

## 函数定义

C函数定义编码风格与声明是不同的（参看[函数声明](#函数声明)部分）。返回类型独占一行，第二行是函数的名称和参数列表；`{`独占第三行。下面是NGINX中的代码示例：

```C
ngx_int_t
ngx_http_compile_complex_value(ngx_http_compile_complex_value_t *ccv)
{
    ...
}
```

注意一点的是：在 `(`前后都没有空格，并且在前三行无任何缩进。

如果参数列表很长，比如超过了80个字符，以不同的行分隔之，每一行前缩进4个空格。示例：

```C
ngx_int_t
ngx_http_complex_value(ngx_http_request_t *r, ngx_http_complex_value_t *val,
    ngx_str_t *value)
{
    ...
}
```

若返回的类型是指针类型，在`*`之前有一个空格，如下所示：

```C
static char *
ngx_http_core_pool_size(ngx_conf_t *cf, void *post, void *data)
{
    ...
}
```

## 局部变量

在[命名规范](#命名规范)部分，要求局部变量使用短名字，比如`ev`、`clcf`等。通常它们放在每个函数定义个开始处，而不是在任何代码块的开始处，除非有助于天使或者其他特殊的需求。同样，函数内的标志符（除了`*`前缀），必须垂直对齐。比如：

```C
    ngx_str_t               *value;
    ngx_uint_t               i;
    ngx_regex_elt_t         *re;
    ngx_regex_compile_t      rc;
    u_char                   errstr[NGX_MAX_CONF_ERRSTR];
```

注意标志符`i`、`re`、`rc` 和 `errstr`是如何垂直对齐，但`*`前缀不包括在内。

可能会出现超长的局部变量名，若是一同对齐，会导致代码超级难看。可以将长局部变量名与其他局部变量之间插入空行分隔开来。此时，两组标志符，没有必要垂直对齐。示例如下：

```C
static char *
ngx_http_core_open_file_cache(ngx_conf_t *cf, ngx_command_t *cmd, void
     *conf)
{
    ngx_http_core_loc_conf_t *clcf = conf;

    time_t          inactive;
    ngx_str_t   *value, s;
    ngx_int_t     max;
    ngx_uint_t   i;
    ...
}
```

可以看出`clcf`定义与其他局部变量是分开的，其他的局部变量仍旧垂直对齐的。

在C函数中，局部变量后面与真正执行的代码之间使用空行分隔的。比如：

```C
u_char * ngx_cdecl
ngx_sprintf(u_char *buf, const char *fmt, ...)
{
    u_char   *p;
    va_list   args;

    va_start(args, fmt);
    p = ngx_vslprintf(buf, (void *) -1, fmt, args);
    va_end(args);

    return p;
}
```

正好有一空行在局部变量定义之后。

## 空行的使用

连续的C函数定义、多行全局/静态变量定义和`struct/union/enum`定义必须使用2个空行分隔。下面是连续的C函数定义：

```C
void
foo(void)
{
    /* ... */
}


int
bar(...)
{
    /* ... */
}
```

下面是多个连续的静态变量定义的例子：

```C
static ngx_conf_bitmask_t  ngx_http_core_keepalive_disable[] = {
    ...
    { ngx_null_string, 0 }
};


static ngx_path_init_t  ngx_http_client_temp_path = {
    ngx_string(NGX_HTTP_CLIENT_TEMP_PATH), { 0, 0, 0 }
};
```

仅有一行变量定义必须组织在一起，比如：

```C
static ngx_str_t  ngx_http_gzip_no_cache = ngx_string("no-cache");
static ngx_str_t  ngx_http_gzip_no_store = ngx_string("no-store");
static ngx_str_t  ngx_http_gzip_private = ngx_string("private");
```

下面是多个结构体的定义：

```C
struct ngx_http_log_ctx_s {
    ngx_connection_t    *connection;
    ngx_http_request_t  *request;
    ngx_http_request_t  *current_request;
};


struct ngx_http_chunked_s {
    ngx_uint_t           state;
    off_t                size;
    off_t                length;
};


typedef struct {
    ngx_uint_t           http_version;
    ngx_uint_t           code;
    ngx_uint_t           count;
    u_char              *start;
    u_char              *end;
} ngx_http_status_t;
```

所有的都是以2个空行分隔的。

若不同类型的顶层对象定义也是需要用2行空行分隔，比如：

```C
#if (NGX_HTTP_DEGRADATION)
ngx_uint_t  ngx_http_degraded(ngx_http_request_t *);
#endif


extern ngx_module_t  ngx_http_module;
```

在全局变量声明后面使用2行空行分隔静态函数声明。

多个C函数声明不比适用两行空行彼此分隔，如下所示：

```C
ngx_int_t ngx_http_discard_request_body(ngx_http_request_t *r);
void ngx_http_discarded_request_body_handler(ngx_http_request_t *r);
void ngx_http_block_reading(ngx_http_request_t *r);
void ngx_http_test_reading(ngx_http_request_t *r);
```

甚至在跨越多行的情况下也是如此。比如：

```C
char *ngx_http_merge_types(ngx_conf_t *cf, ngx_array_t **keys,
    ngx_hash_t *types_hash, ngx_array_t **prev_keys,
    ngx_hash_t *prev_types_hash, ngx_str_t *default_types);
ngx_int_t ngx_http_set_default_types(ngx_conf_t *cf, ngx_array_t **types,
    ngx_str_t *default_type);
```

有时候，我们需要根据语意将函数声明分组，然后使用2行空行分隔，这样有助于代码可读性，比如：

```C
ngx_int_t ngx_http_send_header(ngx_http_request_t *r);
ngx_int_t ngx_http_special_response_handler(ngx_http_request_t *r,
    ngx_int_t error);
ngx_int_t ngx_http_filter_finalize_request(ngx_http_request_t *r,
    ngx_module_t *m, ngx_int_t error);
void ngx_http_clean_header(ngx_http_request_t *r);


ngx_int_t ngx_http_discard_request_body(ngx_http_request_t *r);
void ngx_http_discarded_request_body_handler(ngx_http_request_t *r);
void ngx_http_block_reading(ngx_http_request_t *r);
void ngx_http_test_reading(ngx_http_request_t *r);
```

前面一组基本上是响应头部的函数声明，后面一组是请求体相关的函数声明。

## 类型转换

在C语言中将`void`指针（`void *`）赋值给非`void`指针并没有要求显示转换。NGINX中的编码风格也是如此。比如：

```C
char *
ngx_http_types_slot(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    char  *p = conf;
    ...
}
```

示例中`conf`变量是`void`指针类型，将其赋值给一个`char *`局部变量，并没有使用任何显示类型转换。

确实需要显示类型转换的时候，确保在目标指针类型名字的第一个`*`前加一个空格
，同样在`)`后面加入一个空格。比如：

`*types = (void *) -1;`

示例中在`*)`之前有一个空格，同样在`)`后面也有一个。此规则同样适用于将一个值进行类型转换。比如：

```C
if ((size_t) (last - buf) < len) {
    ...
}
```

或者多个连续的类型转换：

`aio->aiocb.aio_data = (uint64_t) (uintptr_t) ev;`

注意`(uint64_t)` 与 `(uintptr_t)`之间的空格，还有在`(uintptr_t)`之后的空格。

## `If`语句

NGINX 适用的`if` 语句与C语言中的编码风格是一样的。

首先，在 `if`后面必须有一个空格；再者`()`与`{`之间有空格。如下：

```C
if (a > 3) {
    ...
}
```

可以看出，在 `if`后面有一个空格；在 `{`之前有一个空格。然而在`(`右边和`)`左边是没有空格。此外，
`{`与`if`关键字在同一行，除非此行超过了80个字符。若是则，我们需要将其分隔多行，`{`独立成行。
如下面的例子：

```C
        if (ngx_http_set_default_types(cf, prev_keys, default_types)
            != NGX_OK)
        {
            return NGX_CONF_ERROR;
        }
```

注意 `!= OK` 与`if` 语句的条件部分对齐（不包括 `(`）。

若逻辑操作符在很长的条件语句中，需要确保连接逻辑操作符在后续行开头处，并且缩进需体现出条件的嵌套结构。如下：

```C
        if (file->use_event
            || (file->event == NULL
                && (of->uniq == 0 || of->uniq == file->uniq)
                && now - file->created < of->valid
#if (NGX_HAVE_OPENAT)
                && of->disable_symlinks == file->disable_symlinks
                && of->disable_symlinks_from == file->disable_symlinks_from
#endif
            ))
        {
            ...
        }
```

我们可以忽略中间的宏指令，它们并不是`if` 语句本身的编码风格。

若在`if` 语句块后有其他的语句，通常会在其后空一行。比如：

```C
        if (rc != NGX_OK && (of->err == 0 || !of->errors)) {
            goto failed;
        }

        if (of->is_dir) {
            ...
        }
```

注意空行是如何分离`if` 语句块的，或者后面跟着其他的语句：

```C
        if (file->is_dir) {

            /*
             * chances that directory became file are very small
             * so test_dir flag allows to use a single syscall
             * in ngx_file_info() instead of three syscalls
             */

            of->test_dir = 1;
        }

        of->fd = file->fd;
        of->uniq = file->uniq;
```

类似的，常常会在`if` 语句前有一空行。比如：

```C
        rc = ngx_open_and_stat_file(name, of, pool->log);

        if (rc != NGX_OK && (of->err == 0 || !of->errors)) {
            goto failed;
        }
```

在代码块之间使用空白符，会让代码不那么臃肿。同样也适用于`while`、`for` 等语句。

尽管只有单条件的`If` 语句，也需要使用 `{}`，比如：

```C
        if (file->is_dir || file->err) {
            goto update;
        }
```

在例子中绝对不能省略`{}`,尽管标准C语言是允许这么做的。

## `else`语句

当`if` 语句有`else`分支的时，相关的语句也必须使用`{}`包裹。而且`} else {`之前有一空行。如下所示：

```C
    if (of->disable_symlinks == NGX_DISABLE_SYMLINKS_NOTOWNER
        && !(create & (NGX_FILE_CREATE_OR_OPEN|NGX_FILE_TRUNCATE)))
    {
        fd = ngx_openat_file_owner(at_fd, p, mode, create, access, log);

    } else {
        fd = ngx_openat_file(at_fd, p, mode|NGX_FILE_NOFOLLOW, create, access);
    }
```

注意`} else {`在同一行，并且在其上方有一空行。

## `For`语句

`for` 语句的编码风格在很多地方与[`If`语句](#If语句)部分描述的`if` 语句相似。在`for`关键字之后和`{`之前有空格。另外，语句块需要用`{}`包裹。再者就是在`for`的条件部分中的`;`后面有空格。下面的例子展示上述的要求：

```C
for (i = 0; i < size; i++) {
    ...
}
```

一个特殊情况是无限循环，在NGINX中通常是如下编码的：

```C
    for ( ;; ) {
        ...
    }
```

或者`for` 语句条件部分包括`,`:

```C
    for (i = 0, n = 2; n < cf->args->nelts; i++, n++) {
        ...
    }
```

或者单独忽略循环条件：

```C
    for (p = pool, n = pool->d.next; /* void */; p = n, n = n->d.next) {
        ...
    }
```

## `While`语句

`while`语句的编码风格在很多地方与[`If`语句](#If语句)部分描述的`if` 语句相似。在`while`关键字之后和`{`之前有空格。另外，语句块需要用`{}`包裹。如下：

```C
    while (log->next) {
        if (new_log->log_level > log->next->log_level) {
            new_log->next = log->next;
            log->next = new_log;
            return;
        }

        log = log->next;
    }
```

`Do-while` 语句与之类似:

```C
    do {
        p = h2c->state.handler(h2c, p, end);

        if (p == NULL) {
            return;
        }

    } while (p != end);
```

注意在`do`和`{`之间存在一个空格，在`while`的前后也是存在的。

## `Switch`语句

`switch`语句的编码风格在很多地方与[`If`语句](#If语句)部分描述的`if` 语句相似。在`switch`关键字之后和`{`之前有空格。另外，语句块需要用`{}`包裹。如下：

```C
    switch (unit) {
    case 'K':
    case 'k':
        len--;
        max = NGX_MAX_SIZE_T_VALUE / 1024;
        scale = 1024;
        break;

    case 'M':
    case 'm':
        len--;
        max = NGX_MAX_SIZE_T_VALUE / (1024 * 1024);
        scale = 1024 * 1024;
        break;

    default:
        max = NGX_MAX_SIZE_T_VALUE;
        scale = 1;
    }
```

注意`case`标签与`switch`关键字是垂直对齐的。

有时，会在第一个`case`标签前加入一空行，如下：

```C
    switch (c->log_error) {

    case NGX_ERROR_IGNORE_EINVAL:
    case NGX_ERROR_IGNORE_ECONNRESET:
    case NGX_ERROR_INFO:
        level = NGX_LOG_INFO;
        break;

    default:
        level = NGX_LOG_ERR;
    }
```

## 分配内存的错误处理

NGINX中有一个很好习惯就是检查动态内存申请的错误。任何地方都如下所示处理：

```C
    sa = ngx_palloc(cf->pool, socklen);
    if (sa == NULL) {
        return NULL;
    }
```

上面的两条语句使用很频繁，通常不会在申请语句和`if`语句之间加入空行。

确保你从来不会遗漏检查类似动态申请内存。

## 函数调用

C函数调用不需要在参数列表的`(`和`)`前后加入空格，如下所示：

`sa = ngx_palloc(cf->pool, socklen);`

若函数调用超过了80个字符，需要将参数列表独立成行，子行中的参数需要与首行参数垂直对齐。如下：

```C
        buf->pos = ngx_slprintf(buf->start, buf->end, "MEMLOG %uz %V:%ui%N",
                                size, &cf->conf_file->file.name,
                                cf->conf_file->line);
```

## 宏

宏定义要求在`#define`后有一个空格，而在定义体前面需至少2个空格。如下：

`#define F(x, y, z)  ((z) ^ ((x) & ((y) ^ (z))))`

有时候定义体前需要更多的空格，出于多个相关宏定义之间对齐。如下：

```C
#define NGX_RESOLVE_A         1
#define NGX_RESOLVE_CNAME     5
#define NGX_RESOLVE_PTR       12
#define NGX_RESOLVE_MX        15
#define NGX_RESOLVE_TXT       16
#define NGX_RESOLVE_AAAA      28
#define NGX_RESOLVE_SRV       33
#define NGX_RESOLVE_DNAME     39
#define NGX_RESOLVE_FORMERR   1
#define NGX_RESOLVE_SERVFAIL  2
```

对于展开多行的宏定义，需要对齐续行符`\`。如下：

```C
#define ngx_conf_init_value(conf, default)
\
    if (conf == NGX_CONF_UNSET) {                                            \
        conf = default;                                                      \
    }
```

我们建议将`\`放在第78列，尽管NGINX前后不一致。

## 全局或静态变量

对于局部变量、顶部的静态变量的定义和声明，需要在类型描述符与变量名之间加入至少两个空格（包括多个前导`*`修饰）

下面是一些示例：

```C
ngx_uint_t   ngx_http_max_module;


ngx_http_output_header_filter_pt  ngx_http_top_header_filter;
ngx_http_output_body_filter_pt    ngx_http_top_body_filter;
ngx_http_request_body_filter_pt   ngx_http_top_request_body_filter;
```

同样地对于带有初始化的变量定义也是如此，如下：

```C
ngx_str_t  ngx_http_html_default_types[] = {
    ngx_string("text/html"),
    ngx_null_string
};
```

## 操作符

### 二元操作符

很多二元操作符前后需要一个空格。比如，算术运算符、位运算符、关系运算符合逻辑运算符。如下：

 `yday = days - (365 * year + year / 4 - year / 100 + year / 400);`

还有，

`if (*p >= '0' && *p <= '9') {`

对于结构体/联合体成员操作符 `->` 和 `.`，在其前后都没有空格。比如：

`ls = cycle->listening.elts;`

对于`,`操作符，在其后面需要有一个空格，而不是前面：

```C
for (p = pool, n = pool->d.next; /* void */; p = n, n = n->d.next) {
```

NGINX中通常避免使用`,`操作符，除非在 `for`语句中声明多个同样类型的变量。在其他情况，最好将`,`表达式
分成多条语句。

### 一元操作符

通常C中得一元操作符前后都不会放入任何空格。如下：

```C
for (p = salt; *p && *p != '$' && p < last; p++) { /* void */ }
#define SET(n)      (*(uint32_t *) &p[n * 4])
```

注意，在`*`和`&`前后都没放入任何空白符（在第二例子中`&`的前面加入了空格，是因为使用了类型转换，更多地可以参考[类型转换][#类型转换]部分）。

对于后缀操作符同样适用：

`for (value = 0; n--; line++) {`

### 三元操作符

三元操作符要求在其前后使用空白符，如同二元操作符一样。比如：

`node = (rc < 0) ? node->left : node->right;`

从上面的例子可以看出，当三元操作中的条件部分是一个表达式时，可以加入`()`，尽管不是必须的。

## 结构体/联合体/枚举 定义

结构体、联合体和枚举定义的风格是相似的，都需要将标志符垂直对齐，与在[局部变量](#局部变量)中说明的局部变量对齐类似。我们列举来自
NGINX核心代码的例子来说明：

```C
typedef struct {
    ngx_uint_t           http_version;
    ngx_uint_t           code;
    ngx_uint_t           count;
    u_char              *start;
    u_char              *end;
} ngx_http_status_t;
```

与局部变量的定义一样，我们同样需要使用空行分开组字段，如下所示：

```C
struct ngx_http_request_s {
    uint32_t                          signature;         /* "HTTP" */

    ngx_connection_t                 *connection;

    void                            **ctx;
    void                            **main_conf;
    void                            **srv_conf;
    void                            **loc_conf;

    ngx_http_event_handler_pt         read_event_handler;
    ngx_http_event_handler_pt         write_event_handler;
    ...
};
```

在此例中，每一组成员变量都是垂直对齐的，但不要求所有的组都一同对齐（尽管我们可以这么做，如上述例子一般）。

联合体也是类似的：

```C
typedef union epoll_data {
    void         *ptr;
    int           fd;
    uint32_t      u32;
    uint64_t      u64;
} epoll_data_t
```

下面是枚举类型：

```C
typedef enum {
    NGX_HTTP_INITING_REQUEST_STATE = 0,
    NGX_HTTP_READING_REQUEST_STATE,
    NGX_HTTP_PROCESS_REQUEST_STATE,

    NGX_HTTP_CONNECT_UPSTREAM_STATE,
    NGX_HTTP_WRITING_UPSTREAM_STATE,
    NGX_HTTP_READING_UPSTREAM_STATE,

    NGX_HTTP_WRITING_REQUEST_STATE,
    NGX_HTTP_LINGERING_CLOSE_STATE,
    NGX_HTTP_KEEPALIVE_STATE
} ngx_http_state_e;
```

## `Typedef` 定义

类似于[宏](#宏)，`typedef`定义同样需要在定义体的前面至少有2个空格，（通常就2个空格）
`typedef u_int  aio_context_t;`

在将`typedef`定义分组的时候可以使用多于2个空格将其垂直对齐，这样可以代码更为美观。

```C
typedef struct ngx_module_s          ngx_module_t;
typedef struct ngx_conf_s            ngx_conf_t;
typedef struct ngx_cycle_s           ngx_cycle_t;
typedef struct ngx_pool_s            ngx_pool_t;
typedef struct ngx_chain_s           ngx_chain_t;
typedef struct ngx_log_s             ngx_log_t;
typedef struct ngx_open_file_s       ngx_open_file_t;
```

## 工具

OpenResty团队维护[ngx-releng](https://github.com/openresty/openresty-devel-utils/blob/master/ngx-releng)工具。它静态地扫描当前C文件，校验本文档所覆盖的风格，但不全部。`ngx-releng是OpenResty核心开发的必选工具，也有助于
NGINX模块开发者或者NGINX核心开发者。我们会持续增加更多地校验功能，同时我们也期待您的加入。

clang静态代码分析器对于获取代码隐匿问题是一个好帮手，你可以开启优化配置标志以编译一切。

很多编辑器提供了高亮后者自动截除行尾的空白符，也可以将制表符扩展成空格。比如，在`vim`中我们可以将下面的
配置放入到 `~/.vimrc`，可以标亮任何行尾空白符：

```Shell
highlight WhiteSpaceEOL ctermbg=darkgreen guibg=lightgreen
match WhiteSpaceEOL /\s$/
autocmd WinEnter * match WhiteSpaceEOL /\s$/
```

或者设置一组简便的属性：

```Shell
set expandtab
set shiftwidth=4
set softtabstop=4
set tabstop=4
```

## `Goto`语句和代码标签

NGINX中使用`goto` 语句进行错误处理是十分明智，这是对臭名昭著的`goto`语句使用比较好的场景。许多非资深的C语言程序员对于`goto`语句使用存在恐慌，这其实是不公平的。（PS：Dijkstra大神的[Go To Statement Considered Harmful](http://www.u.arizona.edu/~rubinson/copyright_violations/Go_To_Considered_Harmful.html)，然而在《代码大全》一书中：

> 90%情况下用goto都是错的，但少数情况goto确实有效（比如在错误处理中的多重判断，又如两个条件判断和一个else子句的情况），是解决问 题的合理办法。这时候用goto无妨，但应该加注释说明理由。

）。使用`goto` 语句往回跳转是十分糟糕的，其他是可以的，特别是错误处理。NGINX要求代码标签包裹在空行中，比如：

```C
        p = ngx_pnalloc(pool, len);
        if (p == NULL) {
            goto failed;
        }

        ...

        i++;
    }

    freeaddrinfo(res);
    return NGX_OK;

failed:

    freeaddrinfo(res);
    return NGX_ERROR;
```

## 校验空指针

在NGINX中，常常使用 `p == NULL`，而不是 `!p` 检查一个指针的值是否为`NULL`。只要校验指针是否为`NULL`的地方就遵守此约定。同理，推荐使用 `p != NULL`而不是 `p` 检验指针的值是非`NULL`，但是有时候使用 `p`也是可以的。

下面是一些例子：

```C
if (addrs != NULL) {
if (name == NULL) {
```

检验 `NULL`通常清楚的表明了检查值属性，并且提高代码的可读性。

## 作者

本指南由OpenResty的创建者章亦春撰写。

## 反馈与建议

有任何反馈或者建议都可以发送邮件<a href="mailto:yichun@openresty.com">章亦春</a>。