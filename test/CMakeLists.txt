# Testing support
find_program(PG_REGRESS pg_regress
  HINTS
  "${PG_PKGLIBDIR}/pgxs/src/test/regress/"
  REQUIRED)

message(STATUS "Using pg_regress ${PG_REGRESS}")

set(TEST_ROLE_SUPERUSER super_user)
set(TEST_ROLE_DEFAULT_PERM_USER default_perm_user)
set(TEST_ROLE_DEFAULT_PERM_USER_2 default_perm_user_2)

# Basic connection info for test instance
set(TEST_PGPORT_LOCAL 5432 CACHE STRING "The port of a running PostgreSQL instance")
set(TEST_PGHOST localhost CACHE STRING "The hostname of a running PostgreSQL instance")
set(TEST_PGUSER ${TEST_ROLE_DEFAULT_PERM_USER} CACHE STRING "The PostgreSQL test user")
set(TEST_DBNAME single CACHE STRING "The database name to use for tests")
set(TEST_PGPORT_TEMP_INSTANCE 15432 CACHE STRING "The port to run a temporary test PostgreSQL instance on")
set(TEST_SCHEDULE ${CMAKE_CURRENT_BINARY_DIR}/test_schedule CACHE STRING "The test schedule to run")
set(TEST_INPUT_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(TEST_OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(TEST_CLUSTER ${TEST_OUTPUT_DIR}/testcluster)

set(PG_REGRESS_OPTS_BASE
  --host=${TEST_PGHOST}
  --load-language=plpgsql
  --dlpath=${PROJECT_BINARY_DIR}/src)

set(PG_REGRESS_OPTS_EXTRA
  --create-role=${TEST_ROLE_SUPERUSER},${TEST_ROLE_DEFAULT_PERM_USER},${TEST_ROLE_DEFAULT_PERM_USER_2}
  --dbname=${TEST_DBNAME}
  --launcher=${TEST_INPUT_DIR}/runner.sh)

set(PG_REGRESS_OPTS_INOUT
  --inputdir=${TEST_INPUT_DIR}
  --outputdir=${TEST_OUTPUT_DIR})

set(PG_REGRESS_OPTS_TEMP_INSTANCE
  --port=${TEST_PGPORT_TEMP_INSTANCE}
  --temp-instance=${TEST_CLUSTER}
  --temp-config=${TEST_INPUT_DIR}/postgresql.conf)

set(PG_REGRESS_OPTS_LOCAL_INSTANCE
  --port=${TEST_PGPORT_LOCAL})

set(PG_REGRESS_ENV
  TEST_PGUSER=${TEST_PGUSER}

  TEST_ROLE_SUPERUSER=${TEST_ROLE_SUPERUSER}
  TEST_ROLE_DEFAULT_PERM_USER=${TEST_ROLE_DEFAULT_PERM_USER}
  TEST_ROLE_DEFAULT_PERM_USER_2=${TEST_ROLE_DEFAULT_PERM_USER_2}
  TEST_DBNAME=${TEST_DBNAME}
  TEST_INPUT_DIR=${TEST_INPUT_DIR}
  TEST_OUTPUT_DIR=${TEST_OUTPUT_DIR}
  TEST_SCHEDULE=${TEST_SCHEDULE}
  PG_BINDIR=${PG_BINDIR}
  PG_REGRESS=${PG_REGRESS})

# installcheck starts up new temporary instances for testing code
add_custom_target(installcheck
  COMMAND ${CMAKE_COMMAND} -E env
  ${PG_REGRESS_ENV}
  ${CMAKE_CURRENT_SOURCE_DIR}/pg_regress.sh
  ${PG_REGRESS_OPTS_BASE}
  ${PG_REGRESS_OPTS_EXTRA}
  ${PG_REGRESS_OPTS_INOUT}
  ${PG_REGRESS_OPTS_TEMP_INSTANCE}
  USES_TERMINAL)

# installchecklocal tests against an existing postgres instance
add_custom_target(installchecklocal
  COMMAND ${CMAKE_COMMAND} -E env
  ${PG_REGRESS_ENV}
  ${CMAKE_CURRENT_SOURCE_DIR}/pg_regress.sh
  ${PG_REGRESS_OPTS_BASE}
  ${PG_REGRESS_OPTS_EXTRA}
  ${PG_REGRESS_OPTS_INOUT}
  ${PG_REGRESS_OPTS_LOCAL_INSTANCE}
  USES_TERMINAL)

add_subdirectory(sql)

if (PG_SOURCE_DIR)
  add_subdirectory(pgtest)
endif (PG_SOURCE_DIR)

if (CMAKE_BUILD_TYPE MATCHES Debug)
  add_subdirectory(src)
endif (CMAKE_BUILD_TYPE MATCHES Debug)
