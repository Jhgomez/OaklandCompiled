## Description

This is a copy of the [original repo](https://github.com/Jhgomez/CompilerFrontend/tree/main/ProjectoParser/assembly). Project 2 is a compiler, which is a type of compiler that creates native executables. The language compiles to 32-bit RISCV assembly, 
the lexical and syntax analyzer live inside the parser generated with peggyJs. The project follows [this](./Enunciado.pdf) specification,
The parser generates an AST, each node is then interpreted as specified in the documentation and we generate RISC-V instructions/code, 
the generated RISC-V code was then executed with a RISC-V simulator, during the development we used different simulators like 
[ripes](https://ripes.me/), [simriscv](https://simriscv.github.io/) and finally we ended up with [rars](https://github.com/TheThirdOne/rars),
ripes was a pretty nice tool due to the user interface it helped visualize/debug the memory usage better, while it was  little 
harder to visualize/debug the memory usage in rars, rars was prefered as it offers hints and descriptions of the risc-v instructions
and also supports floating point instructions/operations. During development we used documentation found in [projectf](https://projectf.io/),
in here you can see a reference to the compiler explorer which is a great tool also, for converting from decimal or text to and 
from HEX I used [this](https://www.rapidtables.com/convert/number/decimal-to-hex.html) calculator, I used this [documentation](https://msyksphinz-self.github.io/riscv-isadoc/html/rvfd.html#)
to get started with floating point instructions, I also learnt about "logical shift" also known as "bit shit" which can be a right 
or left shift, watch this [video](https://www.youtube.com/watch?v=-0FLvN7w3ZI) to learn more. I also learnt that there is at least 
two ways of representing a negative number in low level, we either can use the "IEEE 754" or "two's complement", the first is 
also used to represent floating point numbers. I was also able to understand better what is ASCII and UNICODE, both are
character encoding standards, this means both translate text to/from a decimal value using their own standard, UNICODE has different
implementations such as UTF-8, UTF-16 and UTF-32, each implementation is used in different use cases. In our case to store a
string I got each character UTF-16 decimal value and store them in an array and then store each in the heap as the heap is the dynamic
memory, while the stack is not usually too dynamic. So in the stack I only stored the address of where the first character of the 
string is located and then we know the string ends where we find the first 0 which indicates the end of line/end of string. A similar
process is done with floating point numbers, we take the floating point number and store it inside a JavaScript typed array, but I have
two types arrays working with the same buffer these two types array help mananing/handling binary data in JS and then convert it 
to a HEX string value wich is what risc-v can understand then store it like a decimal value but after that we can only interact
with it using floating point instructions. The idea to be able to manage memory is keep a "stack" which
is some sort or record stored inside a collection of objects in memory, each record has the info about the object that the value in memory represents, like the type, subtype, level(scope), etc. arrays and methods are the closest thing, in this project, that I built that has the idea of how a class should be
represented or stored, I mean we have to define where the properties and values will be stored, for example and array is just and address in the heap that is stored in the stack, this address points to the first item of the array, however at the time I create an array I always store the lenght of the array as some sort of property in the previous position(4 bytes before) in memory, right before the firtst item of the array, 
to be able to read info without messing the stack or heap the pointers are always pointing to the top either of the stack or the heap, the stack pointer is provided by the framework but the heap pointer is just a variable I created that holds the value of an empty/available space in memory right after the last
value that was stored in the heap, the stack pointer behaves differently because "I set it up" this way and that is that the stack pointer is always pointing to the latest value added to the stack, so in order to move for example through an array I create temporary pointers, that means I retrieve the array address which is a heap address stored in the stack and copy it into a register and then start moving it, this helps avoid overwriting data in the stack and/or heap. The Functions behave like the following, first thing we do when 
compiling a function is move the generated code to another "buffer", then leave a space in the stack and the mimic to store the the 
return address and then create the variables that represent the parameters either with an empty space for each or a randon/default value, however be aware a value is always added to the to the mimic of the stack, which again is a collection, then we execute the code just to
generate the instructions that live inside the function, the instrucctions genreated by a function declaration will be written once only
when a function call is found first thing is write the return address and then save the value of the parameters that are being passed as
arguments, then we jump and link and the function is executed. and return when it is finished

Check the entries files to test application, the generated assembly code can be executed in rars, to do that rars will require you to either open an existing file or create a new one, once you have done this, paste the generated code and "compile" with f3 and run it with f5

## Development environment set up

1. Install `node`, to do this I used `nvm`, there is other ways to install `node` anyways. I 
installed `nvm 1.1.12` and used it to install `node v22.5.1`.


2. Make sure `npx` is installed. `npm`, the node package manager, will also be installed; I installed version `10.8.3`. According
to some info `npx`, which is know as the node package runner, will also be installed along `npm` from `npm@5.2.0`
so make sure it is installed with `npx --version` or `where npx`, both commands will only work if
`nodejs` has been added to the environment variable called `path`, this variable specifies a set of 
directories where executable programs are located. `npx` should be installed in the nodejs directory, at least in
windows, so you can check if the executable exists different ways, using the commands above, again make
sure nodejs is already in the `path` or go check in nodejs folder, in windows is inside `program files`. if not installed
just run `npm install -g npx`. `npx` allows us to run node packages without having to download them locally, in
this project we do this since that was a restriction, however this means you could also declare local dependencies, you'd
initiate a file to declare dependencies with `npm init` and define the package we are interested in, in this case it is
**`peggy`**, running it with `npx` means `peggy` is a development only dependency

## Generate Parser

1. Generate the parser from your grammar using peggy, there is different ways to do this. You could
run from the command line directly with `npx peggy ./oakland.pegjs --format=es --dependencies '{"nodes": "./nodes.js"}'`, 
if you get an error try changing peggy version `npx peggy@<version> ./oakland.pegjs --format=es --dependencies '{"nodes": "./nodes.js"}'`,
if you get an error make sure the paths to the `oakland.pegjs` and `nodes.js` files are set correctly. The other options is
to define the arguments in a javascript file like we did in `config.js` then just execute `npx peggy -c ./config.js`, troubleshooting
is the same, check path of `config.js` file and/or try other version of `peggy`. The other option is to generate the parser from peggy's
website, however you can not declare a dependecy like we did before and all the javascript code your parser depends on has
to be declared inside the same `.pegjs` file, javascript code can be declare inside brackets and then in  "Parser variable For CommonJS" enter "Window.peg", download
the ES6 module, ES6 is the new module system.

## Run the app

1. To run the app you should just need to run it from a server, this is a requirement from peggy if its run in this type of setup.
Check the reason in this [documentation](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/set_up_a_local_testing_server#the_problem_with_testing_local_files), 
in this case is because of "They include other files" section. In the same documentation you can see two options to run a server locally
either with `nodejs` or `python`, in our case we used `live server` extension in "VS Code".

2. You can try the [entry files](./entryFIles), copy the generated code and paste in the assembler simulator

## Notes

In the UI I used `ACE` which is a standalone code editor written in JavaScript, I injected it using JSDeliver which is a CDN.
To learn more about how CDNs work look [here](https://www.freecodecamp.org/news/import-javascript-and-css-from-a-public-cdn/),
or search "How to Import JavaScript and CSS from a Public CDN", we did it in one line in our JavaScript script with `import * as aceEditor from 'https://cdn.jsdelivr.net/npm/ace-builds@1.36.2/+esm'`