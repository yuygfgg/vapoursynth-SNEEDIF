project('SNEEDIF', 'cpp',
  default_options : ['buildtype=release', 'b_ndebug=if-release', 'cpp_std=c++20', 'b_lto=true'],
  meson_version : '>=0.51.0',
  version : '8'
)

add_project_arguments('-ffast-math', '-DNNEDI3_DATADIR="@0@"'.format(join_paths(get_option('prefix'), get_option('datadir'), meson.project_name())), language : 'cpp')
add_project_arguments('-Wno-unused-variable', '-Wno-ignored-attributes', '-Wno-sign-compare', '-Wno-narrowing', '-Wno-int-in-bool-context', '-ffast-math', language : 'cpp')

sources = [
  'SNEEDIF/SNEEDIF.cpp',
  'SNEEDIF/NNEDI3.cpp',
  'SNEEDIF/half_float/umHalf.h',
]

vapoursynth_dep = dependency('vapoursynth').partial_dependency(compile_args : true, includes : true)

boost_dep = dependency('boost', modules : ['filesystem'], static : true)

opencl_dep = dependency('OpenCL', required : false)
if not opencl_dep.found()
  opencl_dep = meson.get_compiler('cpp').find_library('OpenCL')
endif

deps = [vapoursynth_dep, boost_dep, opencl_dep]

if host_machine.system() == 'windows'
  deps += [meson.get_compiler('cpp').find_library('winpthread', static: true)]
endif

if host_machine.cpu_family().startswith('x86')
  add_project_arguments('-mfpmath=sse', '-msse2', language : 'cpp')
endif

cpp = meson.get_compiler('cpp')
polly_code = '''
#include <iostream>
int main() {
  return 0;
}
'''

polly_support = true
polly_result = cpp.compiles(polly_code, args: ['-mllvm', '-polly', '-mllvm', '-polly-vectorizer=stripmine', '-mllvm', '-polly-tiling'])

if polly_support and polly_result
  add_project_arguments('-mllvm', '-polly', '-mllvm', '-polly-vectorizer=stripmine', '-mllvm', '-polly-tiling', language: 'cpp')
endif

shared_module('sneedif', sources,
  dependencies : deps,
  gnu_symbol_visibility : 'hidden'
)