
# Headerless C programming with a single macro definition

 Header files in C are painful. They duplicate file count, increase complexity, make refactoring painful. There are solutions to get rid of them. It's possible to use header generators ( https://www.hwaci.com/sw/mkhdr/ - makeheaders ), write you own headerless c dialect with a precompiler like me ( https://github.com/milgra/clc class-c ) or use `"#ifdef FOO_IMPLEMENTATION"` blocks inside header files to define everything in one file but they are unelegant and confusing and have a lot of other problems.

The ultimate solution seems to be using the `__INCLUDE_LEVEL__` preprocessor macro. It's value is zero if we are in a source file that was added directly to the compiler as parameter and greater than zero if we are in a file that was included as a header file from an other file.

So just create a single file, write the header declarations at the top, write the implementation under that and guard the implementation with an `#if __INCLUDE_LEVEL__ == 0` macro and you never have to use header files again unless you need a library interface. You can include all files written this way as header files and add these files as source files to the compiler, everything will work as before.

Example : mtvec.c
```
#ifndef mtvec_h
#define mtvec_h

#include <stdio.h>
#include <stdint.h>

typedef struct mtvec_t mtvec_t;

struct mtvec_t
{
    void** data;
    uint32_t length;
    uint32_t length_real;
};

mtvec_t* mtvec_alloc(void);
void mtvec_dealloc( void* vector );
void mtvec_reset( mtvec_t* vector );

#endif

#if __INCLUDE_LEVEL__ == 0

mtvec_t* mtvec_alloc( )
{
    mtvec_t* vector = mtmem_calloc( sizeof( mtvec_t ) , mtvec_dealloc );
    vector->data = mtmem_calloc( sizeof( void* ) * 10 , NULL );
    vector->length = 0;
    vector->length_real = 10;
    return vector;
}

void mtvec_dealloc( void* pointer )
{
    mtvec_t* vector = pointer;

    for ( uint32_t index = 0 ; index < vector->length ; index++ ) {
    mtmem_release( vector->data[index] );
    }
    mtmem_release( vector->data );
}

void mtvec_reset( mtvec_t* vector )
{
    for ( uint32_t index = 0 ; index < vector->length ; index++ ) mtmem_release( vector->data[index] );

    vector->length = 0;
}

#endif
```

I created/maintain seven applications so far in headerless C and haven't faced any drawbacks yet, feel free to try them :

- MultiMedia File Manager (https://github.com/milgra/mmfm)  
- Visual Music Player (https://github.com/milgra/vmp)  
- Wayland Control Panel (https://github.com/milgra/wcp)  
- Sway Overview (https://github.com/milgra/sov)  
- Cortex (https://github.com/milgra/cortex)  
- Termite (https://github.com/milgra/termite)  
- Brawl (https://github.com/milgra/brawl)  

