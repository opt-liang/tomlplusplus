test_sources = [
	'at_path.cpp',
	'conformance_burntsushi_invalid.cpp',
	'conformance_burntsushi_valid.cpp',
	'conformance_iarna_invalid.cpp',
	'conformance_iarna_valid.cpp',
	'formatters.cpp',
	'impl_toml.cpp',
	'tests.cpp',
	'parsing_floats.cpp',
	'parsing_arrays.cpp',
	'parsing_booleans.cpp',
	'parsing_comments.cpp',
	'parsing_dates_and_times.cpp',
	'parsing_integers.cpp',
	'parsing_key_value_pairs.cpp',
	'parsing_spec_example.cpp',
	'parsing_strings.cpp',
	'parsing_tables.cpp',
	'main.cpp',
	'manipulating_arrays.cpp',
	'manipulating_tables.cpp',
	'manipulating_values.cpp',
	'user_feedback.cpp',
	'using_iterators.cpp',
	'windows_compat.cpp'
]

test_deps = [tomlplusplus_dep]

fs = import('fs')

catch2_dep = dependency('catch2', required: false)
if catch2_dep.found()
	test_deps += catch2_dep
elif fs.exists('..'/'external'/'Catch2'/'single_include')
	catch2_dep = declare_dependency(include_directories: '..'/'external'/'Catch2'/'single_include')
	test_deps += catch2_dep
else
	error('Catch2 is missing! You probably need to fetch submodules ("git submodule update --init --depth 1 external/Catch2"')
endif

has_tl_optional = false
tl_optional_dep = dependency('tl-optional', required: false)
if tl_optional_dep.found()
	test_deps += tl_optional_dep
	has_tl_optional = true
elif fs.exists('..'/'external'/'tloptional'/'include')
	tl_optional_dep = declare_dependency(include_directories: '..'/'external'/'tloptional'/'include')
	test_deps += tl_optional_dep
	has_tl_optional = true
else
	warning('tl-optional not found, skipping its tests')
endif

#######################################################################################################################
# fast math check
#######################################################################################################################

compiler_supports_fast_math_args = []
if compiler.get_id() == 'gcc' or compiler.get_id() == 'clang'
	compiler_supports_fast_math_args += '-ffast-math'
	compiler_supports_fast_math_args += '-ffp-contract=fast'
elif compiler.get_id() == 'msvc' or compiler.get_id() == 'intel-cl'
	compiler_supports_fast_math_args += '/fp:fast'
endif
compiler_supports_fast_math = compiler.links('''
	#include <cmath>
	#include <iostream>
	int main()
	{
		std::cout << std::exp2(2.0) << std::pow(2.0, 3.0) << "\n";
		return 0;
	}
	''',
	name: 'supports fast-math',
	args: compiler_supports_fast_math_args
)

#######################################################################################################################
# do the thing!
#######################################################################################################################

fast_math_modes = [ false, true ]
exception_modes = [ true, false ]
unreleased_feature_modes = [ false, true ]
cpp20_modes = [ false, true ]
test_executables = []
counter = 0
test_base_args = []
test_base_args += universal_args
test_base_args += devel_args

foreach cpp20 : cpp20_modes
	if cpp20 and (not compiler_supports_cpp20 or get_option('compile_library'))
		continue
	endif
	foreach unrel : unreleased_feature_modes
		if unrel and get_option('compile_library')
			continue
		endif
		foreach fast_math : fast_math_modes
			if fast_math and (not compiler_supports_fast_math or get_option('compile_library'))
				continue
			endif
			foreach exceptions : exception_modes
				if (not exceptions) and get_option('compile_library')
					continue
				endif
				test_name = ''
				test_overrides = []
				test_overrides += overrides
				test_args = []
				test_args += test_base_args

				single_header = (counter % 2 == 1)
				tl_optional = (counter % 4 == 2 and exceptions and not get_option('compile_library') and has_tl_optional)
				address_sanitizer = (is_clang and not (single_header or tl_optional or fast_math)) and get_option('asan_tests')
				ub_sanitizer = (is_clang and not (single_header or tl_optional or fast_math)) and get_option('ubsan_tests')

				if cpp20
					test_name = 'cpp20'
					test_overrides += 'cpp_std=none'
					test_args += compiler_supports_cpp20_args
					if compiler_supports_char8
						test_args += compiler_supports_char8_args
					endif
				else
					test_name = 'cpp17'
				endif

				if exceptions
					test_overrides += 'cpp_eh=default'
					test_args += '-DSHOULD_HAVE_EXCEPTIONS=1'
					if is_windows
						test_args += '-D_HAS_EXCEPTIONS=1'
					endif
				else
					test_name = test_name + '_noexcept'
					test_overrides += 'cpp_eh=none'
					test_args += '-DSHOULD_HAVE_EXCEPTIONS=0'
					if is_windows
						test_args += '-D_HAS_EXCEPTIONS=0'
					endif
				endif

				if address_sanitizer or ub_sanitizer
					test_args += [ '-g3' ]
					if address_sanitizer and ub_sanitizer
						test_overrides += 'b_sanitize=address,undefined'
					elif address_sanitizer
						test_overrides += 'b_sanitize=address'
					else
						test_overrides += 'b_sanitize=undefined'
					endif
				endif

				if fast_math
					test_name = test_name + '_fastmath'
					test_args += compiler_supports_fast_math_args
				endif

				if compiler_supports_float16 or compiler_supports_fp16
					if compiler_supports_fp16
						test_args += '-DSHOULD_HAVE_FP16=1'
					endif
					if compiler_supports_float16
						test_args += '-DSHOULD_HAVE_FLOAT16=1'
					endif
				endif
				if compiler_supports_int128
					test_args += '-DSHOULD_HAVE_INT128=1'
				endif
				if compiler_supports_float128
					test_args += '-DSHOULD_HAVE_FLOAT128=1'
				endif

				if unrel
					test_name = test_name + '_unrel'
					test_args += '-DTOML_ENABLE_UNRELEASED_FEATURES=1'
				else
					test_args += '-DTOML_ENABLE_UNRELEASED_FEATURES=0'
				endif

				if single_header
					test_args += '-DUSE_SINGLE_HEADER=1'
				endif
				if tl_optional
					test_args += '-DUSE_TARTANLLAMA_OPTIONAL=1'
					test_name = test_name + '_tlopt'
				endif

				test_executables += [[
					test_name,
					executable(
						test_name,
						test_sources,
						cpp_args: test_args,
						dependencies: test_deps,
						override_options: test_overrides
					)
				]]

				counter = counter + 1

			endforeach # exceptions
		endforeach # fast_math
	endforeach # strict
endforeach # cpp20

test_locales = [
	'C',
	'en_US.utf8',
	'ja_JP.utf8',
	'it_IT.utf8',
	'tr_TR.utf8',
	'fi_FI.utf8',
	'fr_FR.utf8',
	'zh_CN.utf8',
	'de_DE.utf8'
]

foreach executable : test_executables
	foreach locale : test_locales
		test(
			executable[0] + ' (' + locale + ')', # name
			executable[1], # executable object
			env: ['LC_ALL=' + locale],
			workdir: meson.source_root()/'tests'
		)
	endforeach
endforeach


# add the ODR-test build project
executable(
	'odr_test',
	[ 'odr_test_1.cpp', 'odr_test_2.cpp' ],
	cpp_args: test_base_args,
	dependencies: tomlplusplus_dep,
	override_options: overrides
)
