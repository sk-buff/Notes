# kernel module write, compile, install and remove

This document explains the complete process of writing, compiling, installing and removing a kernel module, we will use the following kernel module whose name is `tcpprobe.c` as an example:

```c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>

#define MAX_SYMBOL_LEN	64
static char symbol[MAX_SYMBOL_LEN] = "tcp_sendmsg_locked";
module_param_string(symbol, symbol, sizeof(symbol), 0644);

int no_use_variable=0;
module_param_named(no_use, no_use_variable, int, 0644);
MODULE_PARM_DESC(no_use, "this parameter is useless");

/* For each probe you need to allocate a kprobe structure */
static struct kprobe kp = {
	.symbol_name	= symbol,
};

/* kprobe pre_handler: called just before the probed instruction is executed */
static int handler_pre(struct kprobe *p, struct pt_regs *regs)
{
	pr_info("tcpprobe handler_pre executed, no_use_variable is %d\n", no_use_variable);

	/* A dump_stack() here will give a stack backtrace */
	return 0;
}

/* kprobe post_handler: called after the probed instruction is executed */
static void handler_post(struct kprobe *p, struct pt_regs *regs,
				unsigned long flags)
{
    pr_info("tcpprobe handler_post executed, no_use_variable is %d\n", no_use_variable);
}

/*
 * fault_handler: this is called if an exception is generated for any
 * instruction within the pre- or post-handler, or when Kprobes
 * single-steps the probed instruction.
 */
static int handler_fault(struct kprobe *p, struct pt_regs *regs, int trapnr)
{
	pr_info("tcpprobe handler_fault executed, no_use_variable is %d\n", no_use_variable);
	/* Return 0 because we don't handle the fault. */
	return 0;
}

static int __init kprobe_init(void)
{
	int ret;
	kp.pre_handler = handler_pre;
	kp.post_handler = handler_post;
	kp.fault_handler = handler_fault;

	ret = register_kprobe(&kp);
	if (ret < 0) {
		pr_err("register_kprobe failed, returned %d\n", ret);
		return ret;
	}
	pr_info("Planted kprobe at %p\n", kp.addr);
	return 0;
}

static void __exit kprobe_exit(void)
{
	unregister_kprobe(&kp);
	pr_info("kprobe at %p unregistered\n", kp.addr);
}

module_init(kprobe_init)
module_exit(kprobe_exit)
MODULE_LICENSE("GPL");
```

this program is a skeleton of the `tcpprobe` module written by myself, and I add some meaningless code to make is easier to explain some features of kernel modules. `tcpprobe` uses kprobe (we will not discuss kprobe in this document so you have to learn basic principles of kprobe so that you can understand what will this module do) and will print some information before and after the kernel function `tcp_sendmsg_locked` is called. Now, let's start by learning how a kernel module should be written.

### 1.Writing a kernel module

There're some basic elements in a kernel module, now let's introduce them.

##### 1.1 init and exit function

The function wrapped by the macro `module_init` is the **init** function which will be called when this module is installed to kernel. Similarly, the function wrapped by the macro `module_exit` is the **exit** function which will by called when this module is deleted from the kernel. So it's easy to understand we should implement the core logic of our kernel module in the **init** function and do the cleanup task in the **exit** function. The **init** function is the place where our code creates connections with the kernel, and in most cases we will invoke some kernel APIs to use functions provided by the kernel. Note kernel APIs are different from system calls, kernel APIs provide functions to kernel developers or kernel module developers (some kernel modules are included in the kernel source code, like device driver, but everyone can write kernel modules for their own use, the `tcpprobe` module is an example) while system calls provide functions to OS users. For example, kprobe is a series of kernel APIs instead of system calls, which means you can use kprobe in kernel source code or kernel modules, but you can't not use kprobe in a user space program.

The **init** function of `tcpporbe` is `kprobe_init`, where we initialize a `kprobe` structure named `kp` and register it to the kernel with the kernel API `register_kprobe`. The **exit** function of `tcpprobe` is `kprobe_exit`, where we unregister `kp` from the kernel via the kernel API `unregister_kprobe`.

You may notice the **init** function `kprobe_init` is defined with the macro `__init` **TO DO**

##### 1.2 module parameters

We can define parameters which are passed to the module when it's installed to the kernel. Parameters are defined with some macros including `module_param`, `module_param_named` and `module_param_string`, all these macros are defined in `/module/include/moduleparam.h`. The definition of `module_param_named` is like the following:

```
module_param_named(name, value, type, perm) 
  * @name: a valid C identifier which is the parameter name.
  * @value: the actual lvalue to alter.
  * @type: the type of the parameter
  * @perm: visibility in sysfs
```

We will use the following parameter defined in `tcpprobe` as an example to explain how these four parameters of `module_param_named` are used:

```c
module_param_named(no_use, no_use_variable, int, 0644);
```

 `name` is used to identify the parameter when passing parameters to module at kernel installation period. `type` defines the type of the parameter. For more details let's refer comments in `/module/include/moduleparam.h`:

```
 * The @type is simply pasted to refer to a param_ops_##type and a
 * param_check_##type: for convenience many standard types are provided but
 * you can create your own by defining those variables.
 *
 * Standard types are:
 *	byte, hexint, short, ushort, int, uint, long, ulong
 *	charp: a character pointer
 *	bool: a bool, values 0/1, y/n, Y/N.
 *	invbool: the above, only sense-reversed (N = true)
```

`value` identifies the actual variable defined in the kernel module that is used to store the the value of the corresponding parameter, i.e., when you pass the parameter whose name is `no_use` to `tcpprobe`, the value of `no_use` will be stored in the variable whose name is `no_use_variable`. In other words, accessing `no_use_variable` is how `tcpprobe` actually get the value of the parameter `no_use` that is passed in. Now let's see how to pass parameters when installing a module. When we install a kernel module via `insmod` command (we will discuss `insmod` later, now you just need to know this command is used to install a module to kernel, and we assume that `tcpprobe.c` has been compiled to the object file `tcpprobe.ko`), we can specify the value of a parameter using `parameter=value` as a parameter of the command, just like:

```bash
sudo insmod tcpprobe.ko no_use=9
```

then we can use `dmesg` to show information printed by `pr_info`, and we can see (time info is omitted):

```
......
tcpprobe handler_pre executed, no_use_variable is 9
tcpprobe handler_post executed, no_use_variable is 9
......
```

Unlike calling a function, you don't have to pass every parameter defined in a module when installing it. The corresponding variable to parameter `no_use` is `no_use_variable`, and it is initialized to 0. So if you don't pass the `no_use` parameter when using `insmod`:

```bash
sudo insmod tcpprobe.ko
```

the result of `dmesg` will be like the following:

```
tcpprobe handler_pre executed, no_use_variable is 0
tcpprobe handler_post executed, no_use_variable is 0
```

Now let's talk about the `perm` parameter of `module_param_named`. Each module has a corresponding directory in *sysfs*, the pseudo file system mounted at `/sys`. For example, after installing `tcpprobe`, the directory `/sys/module/tcpprobe` will be created, and we can enter the `/sys/module/tcpprobe/parameters` directory and run `ls` command, then we will see:

```bash
test-server:/sys/module/tcpprobe/parameters$ ls
no_use  symbol
```

As we can see, each parameter defined in a module will create a file with the same name as the parameter in the `/sys/module/module_name/parameters` directory, and the `perm` parameter in `module_param_named` specifies the visibility of these parameter files. In our case, `perm` of both `no_use` and `symbol` (we haven't introduce `module_param_string` yet but it also has the `perm` parameter with the same meaning as `module_param_named`) are `0644`, which means those two files are only writable with root privilege. If you are familiar with Linux files then it's to understand `0666` means every can write those files and `0444` means those files are only readable, and there's a special vale `0` which means this parameter will not create a file in *sysfs*. If we have permissions to write parameter files, we can change values of kernel module variables at runtime. For example, after installing `tcpprobe`, we can use `sudo vim /sys/module/tcpprobe/parameters/no_use` to change the value of `no_use_variable` in `tcpprobe`. If we write 100 to `/sys/module/tcpprobe/parameters/no_use`, we will see the following `dmesg` result:

```
tcpprobe handler_pre executed, no_use_variable is 100
tcpprobe handler_post executed, no_use_variable is 100
```

About kernel module parameters, we can draw the following conclusion:

(1) Parameters are actually variables defined and used by kernel modules.

(2) We can specify the values of those variables at module installation time by passing parameter values.

(3) We can change the values of those variables at module running time using the *sysfs* interface. 



As long as you understand the `module_param_named` macro, it's easy to understand other macros used to define module parameters, and we will introduce some of the briefly.

`module_param` is a special case of `module_param_named` where the name of the parameter is the same as it's corresponding module variable.

```c
#define module_param(name, type, perm)				\
	module_param_named(name, name, type, perm)
```

You may have noticed that `type` in `module_param_named` doesn't contain string (it contains a char pointer type only), because string parameters are defined by `module_param_string(name, string, len, perm)`. We can see how to use `module_param_string` by looking at it's comments:

```
/**
 * module_param_string - a char array parameter
 * @name: the name of the parameter
 * @string: the string variable
 * @len: the maximum length of the string, incl. terminator
 * @perm: visibility in sysfs.
 *
 * This actually copies the string when it's set (unlike type charp).
 * @len is usually just sizeof(string).
 */
```

Just as the comments say, when its values is passed in, a module parameter defined by `module_param_string` will actually copy that value to the `string` variable. In my point of view, this is more convenient when *sysfs* gets involved. For parameters with `charp` type, users have to write a value of a pointer to the parameter file under *sysfs*, which is obscure. But with `module_param_string`, users can write the original string to the parameter file directly, which is more clear and easy to understand.

If you want to pass a list of parameters (they must have the same type so that they can be stored in an array), you may use `module_param_array_named(name, array, type, nump, perm)`, the five parameters means:

```
/**
 * module_param_array_named - renamed parameter which is an array of some type
 * @name: a valid C identifier which is the parameter name
 * @array: the name of the array variable
 * @type: the type, as per module_param()
 * @nump: optional pointer filled in with the number written
 * @perm: visibility in sysfs
 *
 * This exposes a different name than the actual variable name.  See
 * module_param_named() for why this might be necessary.
 */
```

I'll give a simple example to explain how `module_param_array_named` is used, assume we have a module called `testmodule` which contains the following code:

```c
#define MAX_FISH 64

static int fish_array[MAX_FISH];
static int nr_fish;
module_param_array_named(fish, fish_array, int, &nr_fish, 0644);
```

then we can pass a list of parameters separated by commas like:

```bash
sudo insmod testmodule.ko fish=1,2,3
```

and we will get `fish_array[0] = 1`, `fish_array[1] = 2`, `fish_array[2] = 3` and `nr_fish = 3`, we can check the value of `fish` via *sysfs*:

```bash
test-server:~/probe$ cat /sys/module/tcpprobe/parameters/fish
1,2,3
```

we can also change the value of `fish` via *sysfs*, because we the permission of `/sys/module/tcpprobe/parameters/fish` is 0644. Assume we execute `sudo vim /sys/module/tcpprobe/parameters/fish` and replace the content of the file with `4,5`, then we will get `fish_array[0] = 4`, `fish_array[1] = 5`, `fish_array[2] = 3` and `nr_fish = 2`. Note passing *k* values to an array will only influence the first *k* items in that array, so the value of `fish_array[2]` is still 3 after passing new value `4,5` to `testmodule`, but the value of `nr_fish` becomes 2.

Finally, you can document your parameters by using `MODULE_PARM_DESC(_parm, desc)`. For example the information about `no_use` defined by`MODULE_PARM_DESC(no_use, "this parameter is useless")` in `tcpprobe` can be printed by executing `modinfo` command:

 ```bash
test-server:~/probe$ modinfo tcpprobe.ko
filename:       /home/liuchang/probe/tcpprobe.ko
license:        GPL
srcversion:     04B8F82437CD5C789307E7D
depends:
retpoline:      Y
name:           tcpprobe
vermagic:       4.19.87-041987-generic SMP mod_unload
parm:           symbol:string
parm:           no_use:this parameter is useless (int)
 ```

##### 1.3 ways to output information



