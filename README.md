<img  align="left" src="https://github.com/BelalElhawary/UnityLua/blob/main/unity_lua_logo.png " alt="image" width="80" height="80">

# UnityLua
Lua scripting language support for the Unity game engine 

* [About](#about-unitylua)
* [HelloWorld](#helloworld)
* [Supported Platforms](#supported-platforms)
* [Performance](#performance)

## About UnityLua
UnityLua uses platform-specific pre-compiled lua library with [KeraLua](https://github.com/NLua/KeraLua) as a c# interface, so you technically use lua native interpreter in your game wither for simplifying the development process or for adding modding support UnityLua got your back.

## HelloWorld
to add UnityLua to your project all you need to do is to import the package from the latest release

first lets create new Lua class
and don't forget to call Close after you finished to free any allocated memory and avoid any wired behavior

``` c#
using KeraLua;

private void RunLua()
{
    Lua lua = new Lua();
    // Here goes your logic
    lua.Close();
}
```
or
``` c#
using(Lua lua = new Lua())
{
    // Here goes your logic
}
```
lua doesn't know about our unity logger so let's start by replacing the lua default print with our own print function
``` c#
// Lua CFunction must be static int and take one argument IntPtr 'the pointer to the lua state'
private static int Print(IntPtr ptr)
{
    // first we get the lua instance from the pointer
    Lua lua = Lua.FromIntPtr(ptr);

    // lua push it's function arguments to the stack by reversing it's order
    // this is meaning the if we get 3 arguments the first one will be -3 the second -2 and so on
    // our function only gets one argument so we will take the -1 (value on the top of the stack) as string (this will convert any bool or number value to string automatically by lua)
    string message = lua.ToString(-1);
    // finally we pass this value to our unity logger
    Debug.Log(message);

    // !! Don't close the instance !!

    // this 0 is the count of our function return values (yes in lua you can return more than one value)
    // in our case we don't return any so we will return 0
    return 0;
}

private void RunLua() 
...
```
after we declare the static function in c# we need to pass it lua (override the existing print)
```c#
// pushing our c# function to lua as c function
lua.PushCFunction(Print);

// now our function live on the top of the stack lets assign it to print
// will set the value on top of stack to the given key
lua.SetGlobal("print")
```
now we have replaced lua default print with our own C# function

to try this out let's run some lua code
```c#
// after lua.SetGlobal("print")

// pass the code to lua to parse and run
// if lua fails to parse the code it will return true
// and pushes the error message to the top of the stack
bool failed = lua.DoString(@"
print('Hello from lua')
");

// if failed we print error message to the console
// all you need to do is to take the stack top value and print it
if(failed)
    Debug.LogError(lua.ToString(-1));

```
what about sending data to lua?
easy task start by pushing a new value (string) to the stack
```c#
// after lua.SetGlobal("print")

// will push string to the stack
lua.PushString("Hello from C#"); 
```

then set it's value to the global variable `CsMessage`
``` c#
// will set the value on top of the stack to the global variable 'CsMessage'
lua.SetGlobal("CsMessage"); 
```
now lets modify our program to print our message
``` c#
bool failed = lua.DoString(@"
print('Hello from lua')
print(CsMessage)
");
```

congratulations now you have your lua scripting language up and running

this is only small peek of what lua is capable of to learn more about how lua  C Api works i recommend watching [Dave Poo series on how bind lua into c++](https://www.youtube.com/watch?v=xrLQ0OXfjaI&list=PLLwK93hM93Z3nhfJyRRWGRXHaXgNX0Itk) it's will explained and easy to understand all lua c api is existing within lua class

if you find this is too complicated we recommend to use UnityLua interface (still in development)

## Supported platforms
🪟 Windows - **x64**, **x86**

📱 Android - **ARMv7**, **AArch64**, **x86**, **x86_64**

🐧 Linux - **x64**

Mac - `working on it`

WebGl - `working on it`

IOS - `working on it`

## Performance
Unity lua is about 70 times faster than moonsharp

| Test          | UnityLua | MoonSharp | Native CSharp |
|---------------|----------|-----------|---------------|
| Fibonacci     | 0.84s    | 62.32s    | 0.07s         |
| Instantiation | 0.39s    | 26.10s    | 0.94s         |
| Binary Trees  | 0.48s    | 40.61s    | 0.46s         |
| Zoo           | 0.31s    | 18.81s    | 0.02s         |