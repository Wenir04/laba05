## Laboratory work V
[![Coverage Status](https://coveralls.io/repos/github/Wenir04/laba05/badge.svg?branch=master)](https://coveralls.io/github/Wenir04/laba05?branch=master)
```
git clone https://github.com/tp-labs/lab05
cd lab05
git remote remove origin
git remote add origin https://github.com/Wenir04/laba05
````
- [x] 1.Создайте `CMakeList.txt` для библиотеки *banking*.
```
nano CMakeLists.txt
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" OFF)

if(BUILD_TESTS)
  add_compile_options(--coverage)
endif()

project (banking)

add_library(banking STATIC ${CMAKE_CURRENT_SOURCE_DIR}/banking/Transaction.cpp ${CMAKE_CURRENT_SOURCE_DIR}/banking/Account.cpp)
target_include_directories(banking PUBLIC
${CMAKE_CURRENT_SOURCE_DIR}/banking )

target_link_libraries(banking gcov)

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB BANKING_TEST_SOURCES tests/*.cpp)
  add_executable(check ${BANKING_TEST_SOURCES})
  target_link_libraries(check banking gtest_main)
  add_test(NAME check COMMAND check)
endif()
```

- [x] 2. Создайте модульные тесты на классы `Transaction` и `Account`
```
mkdir tests
cd tests 
nano test_Account.cpp
include <Account.h>
#include <gtest/gtest.h>

TEST(Account, Banking){
	Account test(0,0);
	ASSERT_EQ(test.GetBalance(), 0);
	ASSERT_THROW(test.ChangeBalance(100), std::runtime_error);
	test.Lock();
	ASSERT_NO_THROW(test.ChangeBalance(100));
	ASSERT_EQ(test.GetBalance(), 100);
	ASSERT_THROW(test.Lock(), std::runtime_error);
	test.Unlock();
	ASSERT_THROW(test.ChangeBalance(100), std::runtime_error);
}

nano test_Transaction.cpp
#include <Account.h>
#include <Transaction.h>
#include <gtest/gtest.h>

TEST(Transaction, Banking){
	Account Alice(0,base_A), Bob(1,base_B);
	Transaction test_tran;
	ASSERT_EQ(test_tran.fee(), 1);
	test_tran.set_fee(base_fee);
	ASSERT_EQ(test_tran.fee(), base_fee);
	ASSERT_THROW(test_tran.Make(Alice, Alice, 1000), std::logic_error);
	ASSERT_THROW(test_tran.Make(Alice, Bob, -50), std::invalid_argument);
	ASSERT_THROW(test_tran.Make(Alice, Bob, 50), std::logic_error);
	if (test_tran.fee()*2-1 >= 100)
		ASSERT_EQ(test_tran.Make(Alice, Bob, test_tran.fee()*2-1), false);
	Alice.Lock();
	ASSERT_THROW(test_tran.Make(Alice, Bob, 1000), std::runtime_error);
	Alice.Unlock();
	ASSERT_EQ(test_tran.Make(Alice, Bob, 1000), true);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
	ASSERT_EQ(test_tran.Make(Alice, Bob, 3900), false);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
}
```

- [x] 3. Настройте сборочную процедуру
```
mkgir .github
cd .github
mkdir workflows 
cd workflows
nano cmake.yml
name: CMake

on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs:
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Adding gtest
    run: git clone https://github.com/google/googletest.git third-party/gtest -b release-1.11.0

  - name: Install lcov
    run: sudo apt-get install -y lcov

  - name: Config banking with tests
    run: cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON

  - name: Build banking
    run: cmake --build ${{github.workspace}}/build

  - name: Run tests
    run: build/check

  - name: Do lcov stuff
    run: lcov -c -d build/CMakeFiles/banking.dir/banking/ --include *.cpp --output-file ./coverage/lcov.info

  - name: Publish to coveralls.io
    uses: coverallsapp/github-action@v1.1.2
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

- [x] 4. Настроим покрытие 
```
mkdir coverage
cd coverage
touch temp.txt
```
На сайте Coveralls.io войдем через гитхаб добавим репозиторий 5 лабы автоматически после пуша пойдет тест
![изображение](https://github.com/Wenir04/laba05/assets/113133600/688369ca-58e0-43cf-9c44-1881a5dbd4e4)
