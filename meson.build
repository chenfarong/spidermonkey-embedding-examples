project('spidermonkey-embedding-examples', 'cpp', version: 'esr60',
    meson_version: '>= 0.43.0', default_options: ['warning_level=3'])

cxx = meson.get_compiler('cpp')

args = ['-std=c++14', '-Wall']

spidermonkey = dependency('mozjs-60')

# Check if SpiderMonkey was compiled with --enable-debug. If this is the case,
# you must compile all your sources with -DDEBUG=1.
# See https://bugzilla.mozilla.org/show_bug.cgi?id=1261161
nondebug_spidermonkey = cxx.compiles('''
#include <js-config.h>
#ifdef JS_DEBUG
#error debug yes, if we did not already error out due to DEBUG not being defined
#endif
''',
    args: args, dependencies: spidermonkey,
    name: 'SpiderMonkey debug check')

if not nondebug_spidermonkey
    args += '-DDEBUG=1'
endif

# Check if a minimal SpiderMonkey program compiles, links, and runs. If not,
# it's most likely the case that SpiderMonkey was configured incorrectly, for
# example by building mozglue as a shared library.
minimal_program = cxx.run('''
#include <js/Initialization.h>
int main(void) {
    if (!JS_Init()) return 1;
    JS_ShutDown();
    return 0;
}
''',
    args: args, dependencies: spidermonkey,
    name: 'SpiderMonkey sanity check')

if not minimal_program.compiled() or minimal_program.returncode() != 0
    error('''
A minimal SpiderMonkey program could not be compiled, linked, or run. Most
likely you should build it with a different configuration. Check the recommended
configuration in this repository.''')
endif

add_project_arguments(args, language: 'cpp')

if cxx.get_id() == 'gcc' or cxx.get_id() == 'clang'
    test_warning_args = [
        # it's useful not to have to initialize all struct fields when defining a
        # JSClass, since some of them are padding.
        '-Wno-missing-field-initializers',

        # These warnings are generated by SpiderMonkey headers.
        '-Wno-non-virtual-dtor', '-Wno-gnu-zero-variadic-macro-arguments',
        '-Wno-nested-anon-types', '-Wno-unused-parameter',
    ]
else
    test_warning_args = []
endif

add_project_arguments(cxx.get_supported_arguments(test_warning_args),
    language: 'cpp')