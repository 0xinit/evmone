# evmone: Fast Ethereum Virtual Machine implementation
# Copyright 2021 The evmone Authors.
# SPDX-License-Identifier: Apache-2.0

set(PREFIX ${PROJECT_NAME}/integration)

get_target_property(EVMONE_LIB_TYPE evmone TYPE)
if(EVMONE_LIB_TYPE STREQUAL SHARED_LIBRARY)

    add_test(NAME ${PREFIX}/histogram COMMAND evmc::tool --vm $<TARGET_FILE:evmone>,histogram run 6000808080800101010200)
    set_tests_properties(
        ${PREFIX}/histogram PROPERTIES PASS_REGULAR_EXPRESSION
        "--- # HISTOGRAM depth=0
opcode,count
STOP,1
ADD,3
MUL,1
PUSH1,1
DUP1,4
")

    add_test(NAME ${PREFIX}/trace COMMAND evmc::tool --vm $<TARGET_FILE:evmone>,trace run 60006002800103)
    set_tests_properties(
        ${PREFIX}/trace PROPERTIES PASS_REGULAR_EXPRESSION
        "\"pc\":0,\"op\":96,\"gas\":\"0xf4240\",\"gasCost\":\"0x3\",\"memSize\":0,\"stack\":\\[\\],\"depth\":1,\"refund\":0,\"opName\":\"PUSH1\"}
{\"pc\":2,\"op\":96,\"gas\":\"0xf423d\",\"gasCost\":\"0x3\",\"memSize\":0,\"stack\":\\[\"0x0\"\\],\"depth\":1,\"refund\":0,\"opName\":\"PUSH1\"}
{\"pc\":4,\"op\":128,\"gas\":\"0xf423a\",\"gasCost\":\"0x3\",\"memSize\":0,\"stack\":\\[\"0x0\",\"0x2\"\\],\"depth\":1,\"refund\":0,\"opName\":\"DUP1\"}
{\"pc\":5,\"op\":1,\"gas\":\"0xf4237\",\"gasCost\":\"0x3\",\"memSize\":0,\"stack\":\\[\"0x0\",\"0x2\",\"0x2\"\\],\"depth\":1,\"refund\":0,\"opName\":\"ADD\"}
{\"pc\":6,\"op\":3,\"gas\":\"0xf4234\",\"gasCost\":\"0x3\",\"memSize\":0,\"stack\":\\[\"0x0\",\"0x4\"\\],\"depth\":1,\"refund\":0,\"opName\":\"SUB\"}
")

    add_test(NAME ${PREFIX}/validate_eof COMMAND evmc::tool --vm $<TARGET_FILE:evmone>,validate_eof run --rev 15 EF0001)
    set_tests_properties(
        ${PREFIX}/validate_eof PROPERTIES PASS_REGULAR_EXPRESSION
        "contract validation failure")
        
    add_test(NAME ${PREFIX}/validate_eof_success COMMAND evmc::tool --vm $<TARGET_FILE:evmone>,validate_eof run --rev 15 EF00010100040200010001FF0000000080000000)
    set_tests_properties(
        ${PREFIX}/validate_eof_success PROPERTIES PASS_REGULAR_EXPRESSION
        "Result:   success")
        
    add_test(NAME ${PREFIX}/validate_eof_create COMMAND evmc::tool --vm $<TARGET_FILE:evmone>,validate_eof run --rev 15 --create EF00010100040200010001FF0000000080000000)
    set_tests_properties(
        ${PREFIX}/validate_eof_create PROPERTIES PASS_REGULAR_EXPRESSION
        "contract validation failure")
        
    add_test(NAME ${PREFIX}/validate_eof_create_success COMMAND evmc::tool --vm $<TARGET_FILE:evmone>,validate_eof run --rev 15 --create EF0001010004020001000403000100000014FF000000008000025F5FEE00EF00010100040200010001FF0000000080000000)
    set_tests_properties(
        ${PREFIX}/validate_eof_create_success PROPERTIES PASS_REGULAR_EXPRESSION
        "Result:   success")
    
endif()

add_subdirectory(eofparse)
add_subdirectory(export)
add_subdirectory(statetest)
add_subdirectory(t8n)

get_directory_property(ALL_TESTS TESTS)
set_tests_properties(${ALL_TESTS} PROPERTIES ENVIRONMENT LLVM_PROFILE_FILE=${CMAKE_BINARY_DIR}/integration-%p.profraw)

