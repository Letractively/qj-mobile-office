# Пример кода #
## Code example ##
```
//contain stat function definition
#include <sys/types.h>
#include <sys/stat.h>

//lua c api definitions
#include "/usr/local/include/luajit-2.0/lua.h"
#include "/usr/local/include/luajit-2.0/lualib.h"
#include "/usr/local/include/luajit-2.0/lauxlib.h"

//check existence function
int file_exists(lua_State *L) {
	//get 1st argument
	const char* name = luaL_checklstring(L, 1, NULL);
	struct stat info;
	//push stat call result
	lua_pushboolean(L, !stat(name, &info));
	//return one result
	return 1;
}
//create name mapping struct array
static const struct luaL_reg name_map[] = {
	//calling exists(x) in lua'll call file_exists(x)
	{"exists", file_exists},
	//array terminator
	{NULL, NULL}
};
//register libqj library
int luaopen_libqj(lua_State *L) {
	luaL_register(L, "libqj", name_map);
	return 1;
}
```

# Компиляция #
## Compile ##
`gcc source.c -shared -fpic -o libqj.so`

# Использование #
## How to use ##
```
require("libqj")
libqj.exists(x)
```