include_guard(GLOBAL)
cmake_minimum_required(VERSION 3.19 FATAL_ERROR)
include(FetchContent)
project("thezhe/osv_scanner_lock")
# Fetch content using `osv-scanner.json`
function(thezhe_osv_scanner_lock errorMessage)
    set(filename "osv-scanner.json")
    # Reconfigure on change
    configure_file("${filename}" ".cache/thezhe/osv_scanner_lock" COPYONLY)
    execute_process(
        COMMAND osv-scanner -v
        RESULT_VARIABLE osv_scanner_missing
        OUTPUT_QUIET ERROR_QUIET)
    if(osv_scanner_missing)
        message(
            WARNING "thezhe_osv_scanner_lock: Cannot find osv-scanner command"
                    ", skipping scan")
    endif()
    # Scan
    if(PROJECT_IS_TOP_LEVEL AND NOT osv_scanner_missing)
        execute_process(
            COMMAND osv-scanner --lockfile osv-scanner:osv-scanner.json
            WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
            RESULT_VARIABLE result
            OUTPUT_VARIABLE output
            ERROR_VARIABLE output
            OUTPUT_STRIP_TRAILING_WHITESPACE ERROR_STRIP_TRAILING_WHITESPACE)
        if(NOT result EQUAL 0)
            set(${errorMessage}
                "${output}"
                PARENT_SCOPE)
            return()
        endif()
    endif()
    # Populate content
    file(READ "${filename}" filestring)
    string(JSON packages GET "${filestring}" "results" 0 "packages")
    string(JSON packages_size LENGTH "${packages}")
    math(EXPR packages_last "${packages_size} - 1")
    # cmake-lint: disable=E1120
    foreach(index RANGE ${packages_last})
        string(JSON package GET "${packages}" "${index}" "package")
        string(JSON name GET "${package}" "name")
        string(JSON commit GET "${package}" "commit")
        string(CONCAT git_repository "https://" "${name}" ".git")
        cmake_path(GET name STEM LAST_ONLY repo_name)
        message(STATUS "thezhe_osv_scanner_lock: Fetching \"${repo_name}\"")
        FetchContent_Declare(
            "${repo_name}"
            GIT_REPOSITORY "${git_repository}"
            GIT_TAG "${commit}"
            GIT_SHALLOW TRUE)
        FetchContent_MakeAvailable("${repo_name}")
    endforeach()
endfunction()
# Test
if(NOT PROJECT_IS_TOP_LEVEL)
    return()
endif()
thezhe_osv_scanner_lock(error_message)
message(STATUS "thezhe_osv_scanner_lock: ${error_message}")
if(NOT error_message)
    message(FATAL_ERROR "thezhe_osv_scanner_lock: Failing test did not fail")
endif()
