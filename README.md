# semantics
Semantic arguments regarding code style and program execution (hmm...)

## Minimising stack exchange; program correctness, code correctness

[Original comment on mattkefford's post](https://github.com/MaJerle/c-code-style/issues/4#issuecomment-3629750546)

[mattkefford](https://github.com/mattkefford) > "... all variables at the top of a block is ... old fashioned."

[MaJerle](https://github.com/MaJerle) > "... very ugly ... unclear ... in the middle of program..."

I believe that context is required before making this decision; and that declarations (and furthermore, initialisations) are better suited closer to where they are used.

Consider the code block at the bottom. Each case results in the call to an inline function; which may or may not declare and initialise it's own set of stack variables. These code blocks get copied and optimised by the compiler; as if each case contained what is contained in the inline function that is called. The names of these functions' stack variables may clash; though, it is inconsequential for the case of an inline function; as they are somehow forbidden from reaching outside their scope ['inline function specifier' at cppreference.com](https://cppreference.com/w/c/language/inline.html): I presume that that is because the variable names could be mangled into some hexadecimal identifier, rather than the literal ASCII. (eg. 23ED4, instead of 'count_symbol_a'). Alternatively, declarations are re-ordered and combined as neccessary. I have not yet dug into the preprocessed code, nor the assembly; in order to know for sure. Regardless, if it turns out that the inline'd functions results in incorrect code, the programmer must declare, optimise and re-order declarations as neccessary.

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

A new issue is that the code is significantly slower; and, you are no longer able to use the message within the preprocessor environment yourself.

### some other header:
```
#define MSG_LEN_INDETERMINATE_TYPE_STRING 39
#define MSG_CODE_INDETERMINATE_TYPE_STRING 32766
extern vec8* message_indeterminate_type_string;
```

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
inline vec8* get_message_delinearized_indeterminate_type_string();

inline vec8* get_message_delinearized_indeterminate_type_string() {
    if (nullptr != message_indeterminate_type_string) {
        return message_indeterminate_type_string;
    }

    vec8opt opts = 0 | VEC_OPT_HEAP_SPREAD;
    ecode e = allocate_vec8(
        &message_indeterminate_type_string,
        MSG_LEN_INDETERMINATE_TYPE_STRING,
        opts
    );
    if (nullptr != message_indeterminate_type_string) {
        /*your choice of ENOMEM handling, or*/ return nullptr;
    }

    //"Type of string could not be determined."
    append_vec8_char(message, 'T');
    append_vec8_char(message, 'y');
    append_vec8_char(message, 'p');
    append_vec8_char(message, 'e');
    append_vec8_char(message, ' ');
    append_vec8_char(message, 'o');
    append_vec8_char(message, 'f');
    append_vec8_char(message, ' ');
    append_vec8_char(message, 's');
    append_vec8_char(message, 't');
    append_vec8_char(message, 'r');
    append_vec8_char(message, 'i');
    append_vec8_char(message, 'n');
    append_vec8_char(message, 'g');
    append_vec8_char(message, ' ');
    append_vec8_char(message, 'c');
    append_vec8_char(message, 'o');
    append_vec8_char(message, 'u');
    append_vec8_char(message, 'l');
    append_vec8_char(message, 'd');
    append_vec8_char(message, ' ');
    append_vec8_char(message, 'n');
    append_vec8_char(message, 'o');
    append_vec8_char(message, 't');
    append_vec8_char(message, ' ');
    append_vec8_char(message, 'b');
    append_vec8_char(message, 'e');
    append_vec8_char(message, '');
    append_vec8_char(message, 'd');
    append_vec8_char(message, 'e');
    append_vec8_char(message, 't');
    append_vec8_char(message, 'e');
    append_vec8_char(message, 'r');
    append_vec8_char(message, 'm');
    append_vec8_char(message, 'i');
    append_vec8_char(message, 'n');
    append_vec8_char(message, 'e');
    append_vec8_char(message, 'd');
    append_vec8_char(message, '.');

    return message;
}
```
