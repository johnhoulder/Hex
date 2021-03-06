NOW THIS IS VERY IMPORTANT. PLEASE READ IT!
===========================================

First up
--------
If you don't know about php classes, go read on them! They allow for much more
complex systems, and make code more compartimentalized. Also, if you don't know
about classes, then you'll think that you can access functions that are defined
in a class without instanciating it before. It doesn't work that way. So please 
read on them (and it helps on future projects as well).

Now for the modules
-------------------
To load and run the various modules, I am thinking of implementing a system of
hooks. Many open-source applications and scripts implemented plugin/module
support through hooks (e.g. Drupal). This means that all the modules will have
to be implemented as classes, hence my first comment.

The system would, first thing, load all the modules from the modules directory.
Loading the modules before connecting to the server means that we can implement
modules to modify the connection behaviour and many other aspects of the system.
Each module file (saved with the **.mod** extension) would contain a class 
(with the same name as the file, so the **reload.mod** module contains the 
`reload` class). The class will need to define a static function named `info` 
which returns an array. That array gives information on the module, such as its
name, version, author, access level, and most importantly its defined hooks. The
list of defined hooks is formatted as such:

    $info = array(
        ... (rest of the information array)
        'hooks' => array(
            'hook_name' => 'name of function to run',
            'other_hook_name' => 'name of function to run',
        ),
    );

The Hooks
---------
Hooks are points at which the core will pause. Load the modules and run the ones
what define that particular hook. That permits greater possibilities in the
modules, instead of simply having modules that parse input and act on it.  
A little gimmick of the module support will be that 2 modules cannot share the
same name, and they are loaded/run in alphabetical order.

The hooks define actions that the core will accomplish. For instance, a hook
might be *on_connect*, *on_message_received*, or *on_message_send*. The list of
hooks is still to be determined, but something along those lines looks right.
Every action that the core accomplishes should have its set of hooks. Modules
that define, in their *info* function a hook are called whenever 

There are 2 kinds of hooks. *pre* hooks and *post* hooks. *pre* hooks are run
prior to the action linked to the hook, so *pre_on_message_send* hooks are run
before the message has actually been sent (the `raw`/`msg` function calls them).
*post* hooks are run after the action linked to the hook as been accomplish, so
*post_on_message_send* hooks are run after the message has been successfully
sent.  
Some actions do not permit the use of *pre*/*post* hook (*on_message_received*
is one), as the message has to be received for that hook to fired: there is no
*pre* state. Those hooks do not have the *pre*/*post* prefix, only the hook
name.

*pre* hooks have a special ability. As soon as a hook returns *false*, processing
of the following hooks and action are stopped. However no error message are
generated. Hooks should implement that functionnality themselves, by
instanciating the `IRCBot_Log` class.  
*post* hooks do not affect the processing of the action, but can stop the
processing of the following hooks by returning *false*.

When calling the function, the hook will give to it 2 pieces of information: its
name and some related data. The name will always be given, but the data is
related to the specific hook.

The Modules
-----------
Modules are defined in **.mod** files, in the **modules** directory. Files with
any other extension will not be loaded by the core, but can be used by your
module internally.

Your module should consist as a main class named exactly as the module file.
IT'S CASE-SENSITIVE! Your module may contain other classes/function, but they
will not be recognized/loaded by the core.

That class should contain at least one function, `info`. It must be a static
function an return an array.  
The class will be instanciated, so you need to have all the other functions as
members of the class. You can also use class variables.  
**N.B.**: one hook cannot appear multiple times or hold an array/list of
functions. If you need to call multiple functions for a hook, create a "master"
function that will call the various other functions as needed.

    <?php
    /* file: modules/example.mod */
    class example {
        public function __construct() { }

        static function info() {
            return array(
                'name' => 'Example Module', //human-readable name
                'author' => 'xav0989', //author, might want to put your IRC handle/username
                'version' => '1.0.0', //version of the module
                'access' => '', //lowest level of rights needed to use module. Implementation unsure
                'hooks' => array(
                    'pre_on_connect' => 'do_this',
                    'on_disconnect' => 'do_that',
                ),
            );
        }

        public function do_this($hook, $data) {
            //do something
            return true;
        }

        public function do_that($hook, $data) {
            //do something else
            return true;
        }
    }

For more information
--------------------
Feel free to contact me:
- Forums: xav0989
- Kayako: Xavier L
- IRC: xav0989 (and related)
- Jabber: xav0989
- email: xav0989@gmail.com

