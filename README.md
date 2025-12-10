# Programming tips / semantics
General advice for programming / software development.

## Minimising stack exchange; program correctness, code correctness

[Original comment on mattkefford's post](https://github.com/MaJerle/c-code-style/issues/4#issuecomment-3629750546)

[mattkefford](https://github.com/mattkefford) > "... all variables at the top of a block is ... old fashioned."

[MaJerle](https://github.com/MaJerle) > "... very ugly ... unclear ... in the middle of program..."

I believe that context is required before making this decision; and that declarations (and furthermore, initialisations) are better suited closer to where they are used.

Consider the code block at the bottom. Each case results in the call to an inline function; which may or may not declare and initialise it's own set of stack variables. These code blocks get copied and optimised by the compiler; as if each case contained what is contained in the inline function that is called. The names of these functions' stack variables may clash; though, it is inconsequential for the case of an inline function; as the function is somehow forbidden from reaching outside it's scope ['inline function specifier' at cppreference.com](https://cppreference.com/w/c/language/inline.html): I presume that it is inconsequential because the variable names could be mangled into some hexadecimal identifier, rather than the literal ASCII. (eg. 23ED4, instead of 'count_symbol_a'). Alternatively, declarations are re-ordered and combined as neccessary. I have not yet dug into the preprocessed code, nor the assembly; in order to know for sure. Regardless, if it turns out that the inline'd functions results in incorrect code, the programmer must declare, optimise and re-order declarations as neccessary.

Also consider the implementation of 'printf' within GNU coreutils: ['printf.c'](https://git.savannah.gnu.org/), or GNU glibc: ['printf.c > __vfprintf_internal.c | glibc-2.42.9000'](https://git.savannah.gnu.org/)
... if you can find them. I have some trouble doing so, myself.

What I see is that the function is polymorphic based on the "print format specification", like '%s', '%u', '%3c'; etc.
In these cases (eg. switch (c) { case 's':... }), the function declares some variables to assist in implementing the functionality... maybe a character count, an intermediate number type when converting from int to long, etc.

If the standard approach is to declare ALL variables at the top, you will find it is far less efficient that declaring things where they are needed: when converting strings in 'printf', ints and longs would be declared when they are simply not used for that run of the function. Splitting code into separate functions is far better for readability, though, results in significantly more stack movement than is neccessary; lastly, inlining functions is the best for this case: but, can result in incorrect code, depending on the compiler that you used.

```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

typedef enum {
    TYPE_STR_UNKNOWN = 0,
    TYPE_STR_FIRST   = 1,
    TYPE_STR_SECOND  = 2,
    TYPE_STR_THIRD   = 3,
    TYPE_STR_FOURTH  = 4,
    TYPE_STR_FIFTH   = 5
} enum_type_str;

// format and print the message to stderr. TODO: console colour.
void log_console_error(char* message, unsigned int message_length);

void log_console_error(char* message, unsigned int message_length) {
    const char* message_prefix = "error: ";
    const unsigned int message_prefix_length = 7;
    char new_message[message_prefix_length + message_length + 1 + 1];
    memcpy(new_message, message_prefix, message_prefix_length);
    memcpy(new_message + message_prefix_length, message, message_length);
    new_message[message_prefix_length + message_length + 1] = '\n';
    new_message[message_prefix_length + message_length + 1 + 1] = '\0';
    fprintf(stderr, "%s", new_message);
    return;
}

// format and print the message to stdout. TODO: console colour.
void log_console_success(char* message, unsigned int message_length);

void log_console_success(char* message, unsigned int message_length) {
    const char* message_prefix = "success: ";
    const unsigned int message_prefix_length = 9;
    char new_message[message_prefix_length + message_length + 1 + 1];
    memcpy(new_message, message_prefix, message_prefix_length);
    memcpy(new_message + message_prefix_length, message, message_length);
    new_message[message_prefix_length + message_length + 1] = '\n';
    new_message[message_prefix_length + message_length + 1 + 1] = '\0';
    fprintf(stdout, "%s", new_message);
    return;
}

// format and print a message to file/s and exit the program.
void handle_exit(char* message, unsigned int message_length, unsigned int exit_code);

void handle_exit(char* message, unsigned int message_length, unsigned int exit_code) {
    if (!exit_code) { log_console_success(message, message_length); }
    else { log_console_error(message, message_length); }
    exit(exit_code);
}

// get the length of a string.
inline void get_string_length(char* str);

inline unsigned int get_string_length(char* str) {
    char c;
    unsigned int len = 0;
    while ('\0' != (c = str[len])) ++len;
    return len;
}

// get the type of a string
inline void get_string_type(char* str, unsigned int str_length);

inline void get_string_type(char* str, unsigned int str_length) {
    //TODO: something to do here...
}

// Do some work on the first type of string.
inline void type_str_first_do_some_work(char* str, unsigned int str_length);

inline void type_str_first_do_some_work(char* str, unsigned int str_length) {
    // implementation defined; count chars, edit a file, send a network request, etc.
    // handle_exit of bad case, or:
    handle_exit("type of string was the first.", 29, 0);
}

// Do some work on the second type of string.
inline void type_str_second_do_some_work(char* str, unsigned int str_length);

inline void type_str_second_do_some_work(char* str, unsigned int str_length) {
    // implementation defined; count chars, edit a file, send a network request, etc.
    // handle_exit of bad case, or:
    handle_exit("type of string was the second.", 30, 0);
}

// Do some work on the third type of string.
inline void type_str_third_do_some_work(char* str, unsigned int str_length);

inline void type_str_third_do_some_work(char* str, unsigned int str_length) {
    // implementation defined; count chars, edit a file, send a network request, etc.
    // handle_exit of bad case, or:
    handle_exit("type of string was the third.", 29, 0);
}

// Do some work on the fourth type of string.
inline void type_str_fourth_do_some_work(char* str, unsigned int str_length);

inline void type_str_fourth_do_some_work(char* str, unsigned int str_length) {
    // implementation defined; count chars, edit a file, send a network request, etc.
    // handle_exit of bad case, or:
    handle_exit("type of string was the fourth.", 30, 0);
}

// Do some work on the fifth type of string.
inline void type_str_fifth_do_some_work(char* str, unsigned int str_length);

inline void type_str_fifth_do_some_work(char* str, unsigned int str_length) {
    // implementation defined; count chars, edit a file, send a network request, etc.
    // handle_exit of bad case, or:
    handle_exit("type of string was the fifth.", 29, 0);
}

int main(int strc, char** strs) {
    if (strc != 1) handle_exit("You must provide a single string.", 33, EINVAL);
    unsigned int str_length = get_string_length(strs[0]);
    if (!str_length) handle_exit("String was empty.", 17, EINVAL);
    enum_type_str type = get_string_type(strs[0]);
    switch (type) {
        case TYPE_STR_FIRST: type_str_first_do_some_work(strs[0], str_length);
            //break; //is technically not neccessary.
        case TYPE_STR_SECOND: type_str_second_do_some_work(strs[0], str_length);
            //break; //is technically not neccessary.
        case TYPE_STR_THIRD: type_str_third_do_some_work(strs[0], str_length);
            //break; //is technically not neccessary.
        case TYPE_STR_FOURTH: type_str_fourth_do_some_work(strs[0], str_length);
            //break; //is technically not neccessary.
        case TYPE_STR_FIFTH: type_str_fifth_do_some_work(strs[0], str_length);
            //break; //is technically not neccessary.
        case TYPE_STR_UNKNOWN: handle_exit("Type of string could not be determined.", 39, EINVAL);
            //break; //is technically not neccessary.
        default: assert(!"String type determination returned an unexpected value...");
           // break; //is technically not neccessary.
    }
}
```

## Reducing the reverse-ability of code via static analysis

In the above example, strings are passed to `void handle_exit(str, strlen)` literally. These would be stored in the binary closely to how they are written, and this would make it fairly easy to understand the flow of execution with a little effort.

A method to harden your program could be:

Call a `vec8* get_message(enum_msg_code message_code)`,
which calls a `inline vec8* get_message_delinearized_MESSAGE_DESCRIPTION_HERE()` function.

Each character of the message is allocated on the heap, and with ASLR; should significantly harden the program against static and (in a way) dynamic analysis.
The strings no longer immediately stand out as steps in the flow of execution, and you retain maintainability.

A new issue is that the code is significantly slower; and, you are no longer able to use the message within the preprocessor environment.

### get_error(enum_msg_code message_code)
```
vec8* get_error(enum_msg_code message_code);

vec8* get_error(enum_msg_code message_code) {
    switch (message_code) 
        case MSG_CODE_INDETERMINATE_TYPE_STRING:
            return get_message_delinearized_indeterminate_type_string();
        ...
        default: assert(!"unhandled case.")
    }
}
```

### get_string_delinearized_indeterminate_type_string()
```
#define MSG_LEN_INDETERMINATE_TYPE_STRING 39
#define MSG_CODE_INDETERMINATE_TYPE_STRING 32766
extern vec8* message_indeterminate_type_string;

inline vec8* get_message_delinearized_indeterminate_type_string();

inline vec8* get_message_delinearized_indeterminate_type_string() {
    if (nullptr != message_indeterminate_type_string) {
        return message_indeterminate_type_string;
    }

    ecode e = allocate_vec8(
        &message_indeterminate_type_string,
        MSG_LEN_INDETERMINATE_TYPE_STRING,
        VEC_OPT_HEAP_SPREAD
    );
    switch (e) {
        case ENOMEM:
            /*your choice of ENOMEM handling, or*/ return nullptr;
        default: assert(!"unhandled case.")
    }
    
    //"Type of string could not be determined."
    append_vec8_char(message_indeterminate_type_string, 'T');
    append_vec8_char(message_indeterminate_type_string, 'y');
    append_vec8_char(message_indeterminate_type_string, 'p');
    append_vec8_char(message_indeterminate_type_string, 'e');
    append_vec8_char(message_indeterminate_type_string, ' ');
    append_vec8_char(message_indeterminate_type_string, 'o');
    append_vec8_char(message_indeterminate_type_string, 'f');
    append_vec8_char(message_indeterminate_type_string, ' ');
    append_vec8_char(message_indeterminate_type_string, 's');
    append_vec8_char(message_indeterminate_type_string, 't');
    append_vec8_char(message_indeterminate_type_string, 'r');
    append_vec8_char(message_indeterminate_type_string, 'i');
    append_vec8_char(message_indeterminate_type_string, 'n');
    append_vec8_char(message_indeterminate_type_string, 'g');
    append_vec8_char(message_indeterminate_type_string, ' ');
    append_vec8_char(message_indeterminate_type_string, 'c');
    append_vec8_char(message_indeterminate_type_string, 'o');
    append_vec8_char(message_indeterminate_type_string, 'u');
    append_vec8_char(message_indeterminate_type_string, 'l');
    append_vec8_char(message_indeterminate_type_string, 'd');
    append_vec8_char(message_indeterminate_type_string, ' ');
    append_vec8_char(message_indeterminate_type_string, 'n');
    append_vec8_char(message_indeterminate_type_string, 'o');
    append_vec8_char(message_indeterminate_type_string, 't');
    append_vec8_char(message_indeterminate_type_string, ' ');
    append_vec8_char(message_indeterminate_type_string, 'b');
    append_vec8_char(message_indeterminate_type_string, 'e');
    append_vec8_char(message_indeterminate_type_string, '');
    append_vec8_char(message_indeterminate_type_string, 'd');
    append_vec8_char(message_indeterminate_type_string, 'e');
    append_vec8_char(message_indeterminate_type_string, 't');
    append_vec8_char(message_indeterminate_type_string, 'e');
    append_vec8_char(message_indeterminate_type_string, 'r');
    append_vec8_char(message_indeterminate_type_string, 'm');
    append_vec8_char(message_indeterminate_type_string, 'i');
    append_vec8_char(message_indeterminate_type_string, 'n');
    append_vec8_char(message_indeterminate_type_string, 'e');
    append_vec8_char(message_indeterminate_type_string, 'd');
    append_vec8_char(message_indeterminate_type_string, '.');

    return message_indeterminate_type_string;
}
```

## and now; for [something completely different](https://www.youtube.com/watch?v=dO_vv3aRZRo):

Randomise your struct layout!

Instead of the following:
```
#define VEC8_INITIAL_SIZE 32
typedef struct {
    u8 size; //of buffer
    char** buf;
    u8 len; //of data
    enum_enc_t enc; //utf-8, ascii, b64, octal, etc. (or mixed)
    enum_vec_opt_t opt; //heap spread, contiguous, reverse, etc.
} vec8_secure;
```

Do this!
```
#define VEC8_INITIAL_SIZE 32
typedef enum : u8 {
    _VEC8_PROP_UNKNOWN = 0,
    VEC8_PROP_SIZE = 1,
    VEC8_PROP_BUF = 2,
    VEC8_PROP_LEN = 3,
    VEC8_PROP_ENC = 4,
    VEC8_PROP_OPT = 5
} enum_vec8_secure_property;

typedef struct {
    u8 idx;
    u8 property;
} vec8_secure_index_member;

typedef ;

typedef struct {
    vec8_secure_index_member* index[] vec8_secure_index;
    void* one;
    void* two;
    void* three;
    void* four;
    void* five;
} vec8_secure;

// create and setup a vec8
vec8* vec8_secure_initialise() {
    //create and populate the property index.
    //initialise each property.
    //return the vec8*.
}

//then, define:
// _vec8_secure_get_size();
// vec8_secure_get_buf();
// vec8_secure_get_len();
// vec8_secure_get_enc();
// vec8_secure_get_opt();
```

If the structure of your memory is hard to guess, then the heap-spread string is encrypted via byte-packing, alternate encoding/s, further obfuscation and indexing, etc...

It can't be hacked!

and after all, that was the point of encryption.

## hardening your compile step:
As during development; YAGNI: you ain't gonna need it.
`gcc` has some cool features to ensure that you're putting out minimal, correct code:

### ! -g
If it's available outside your network; don't include debugging symbols with the `-g` flag: it makes it far too easy to reverse engineer.

## hardening your runtime:
As with websites and internet servers, you should verify shared libraries, too.
`ld` has some cool features to ensure that your code functions correctly:

### -pie / --pic-executable
Analysing a memory dump is far harder with this option. Symbols cannot override each other, and it forces you to consider dependencies on bad programming styles. We're not working for NASA here; there is memory available, and if there isn't, it's just too much in one go: chunk the function's scope. Line by line, byte by byte, or symbol by symbol is better than crashing.

### huge page tables
There was something or other to dicuss about huge page tables and their effect on entropy... 0.0

### ! --no-trampoline
You'll want to trampoline, or you can bounce.

### --forceinteg
A mostly undocumented option. AFAIK, it's like authenticating your kernel on boot, but for shared libraries.
Which leads me to...

## Your development environment
No, not just your text editor.

### Singing your kernel
Sign it. See [redhat](https://www.redhat.com)

### File permissions
Enforce them. See [redhat](https://www.redhat.com) (and SELinux).
You may wish to extend `acl`, and `ext4`.

### Key management
Don't; a computer does it far better. Buy an HSM!

### Access control; general
You may wish to extend `su`.
`unshare` pretty much everything.
If it does something, and it should do "one thing";
it needs a 'namespace', a 'home directory', and a 'cgroup'.

### Key verification
You'll need a paper copy of your Secure Boot keys. If they are altered by Windows Update replacing your BIOS/UEFI binary, you'll never know.
Which leads me into...

## Bad practices in the industry!

What the hell is wrong with sharing a public TLS certificate?
