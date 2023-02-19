# myOwnCompiler
In this project, I'll make a simple compiler using javascript as base, you can visit
https://citw.dev/tutorial/create-your-own-compiler?p=10 to see search source.

# Lexical Analysis
- The code that we receive in the compiler function is a string. Converting a string of one programming language into a string of another programming language is not straightforward. We'll want to convert the input code into a structure that will be easier to work with.
- We'll create a function that takes the input code, which is a string, and break it down into its most basic blocks, the language syntax. The function will return a flat array of the different syntax blocks it detected and their value (array of objects). The method that performs this kind of task is usually called a tokenizer or lexer.
- In this phase we need a token that analyse each character, like letters in phrase
- Let's create a new file called tokenizer.js, require it in our compiler.js, and call it with the input code.
Let's also return the tokens so we could print them out in our index.js file.
And in our index.js file, we'll stringify the output since our tokens are an array of objects.
In our tokenizer.js file, we'll export a function that receives the input code and returns an empty array of tokens.
- We now need to investigate our input string, character by character, and discover the different syntax blocks. We'll create a pointer to the current char, which we'll increase step by step, and iterate our input string.
- We are going to investigate the current char using several conditions.
- If none of our conditions are met, we'll throw an error.
- If we run our program, we'll see the following error.
- If you remember, the ( character is the first character in our input - (add 2 (sub 4 3)). Let's handle it accordingly.
- If the current char is a parenthesis sign, we'll add a new token to our array, with a type of paren and a value of (. We'll increment the current char pointer and call the continue statement, which terminates our while loop's current iteration and continues to the next one.
- If we rerun our program, we'll receive a new error.
The char a is the first letter of our function name add - (add 2 (sub 4 3)). The char a by itself is not an atomic piece of syntax, rather the full name of the function is, and that is what we'll need to collect.
- Since we're building a simple compiler, we can assume that any letter-based value is a function name. We'll define our letter-based regular expression and check if the current char matches it.
- We'll start collecting our chars into a single variable, as long as they're letter-based. Don't forget to change char to a let variable so we'll be able to change it.
- We'll add a new token with a type of name and continue to the next iteration.
- Now when we run our program, we'll receive an error on the whitespace character. That's a good sign, and it means we handled all the letter-based chars correctly.
- The whitespace character is not a syntax in our language. It's just a separator between pieces of syntax, so we can skip it without collecting it. We'll define a whitespace regular expression and handle it in a similar fashion as we did our letter-based characters.
- In the Lexical Analysis phase, we collected all the syntax pieces of our language, but we ignored the relationships between them. In the Syntactic Analysis phase, we'll build a fully formed representation of our program.
- Our goal in this phase is to represent our Lisp code (string) in a tree-like object (object of objects), which holds all the information we need to transform it into another language (or even execute it). This tree-like structure is called an AST - Abstract Syntax Tree.

# Syntactic Analysis

- In the Lexical Analysis phase, we collected all the syntax pieces of our language, but we ignored the relationships between them. In the Syntactic Analysis phase, we'll build a fully formed representation of our program.
- Our goal in this phase is to represent our Lisp code (string) in a tree-like object (object of objects), which holds all the information we need to transform it into another language (or even execute it). This tree-like structure is called an AST - Abstract Syntax Tree.
- We'll start by taking our tokens that we created in the previous step and passing them into a new function called parser, which we import from the (non-existent) parser.js file. Let's also change the return value to the new lispAST variable, so we'll print it and see our results.
Now let's create the parser.js file. It receives the tokens (array of objects) as an argument and returns an AST (object of objects).
- In our AST, each object, or node, will consist of a type and other properties that describe it. Our tree's root node will be of type Program and will have a property named body which is an array, where each member is a different line in our program. Since our program is simple and only has a single line of code, the body array will contain only a single member.
- We now want to iterate our tokens and build a tree structure. We'll create a pointer to the current token and increment it when needed. We'll create a function called walk which will be executed in a recursive manner and place each new node in the correct place in the AST.
- As we did in the Lexical Analysis phase, we'll have a series of conditions to check the token. If the token doesn't match any of the conditions, we will throw an error stating that we could not handle this specific token.
- We'll begin with a simple token type - number. We'll return a new node with the type NumberLiteral (arbitrary name) and our token value. Notice that we don't mutate our AST directly since we don't know where to place the new node. The walk function is initially called from the body property of the root node (line 18) and will be called recursively to place the nodes in the correct place in the tree.
- We now want to handle the function calling part of our sample code. If you remember, in Lisp, the opening parenthesis sign (defines a function call. We'll look for that in our next condition. Since both our opening and closing parenthesis signs have the same token type - paren, we'll need to differentiate between them based on the value.
- When we find the open parenthesis token, we'll create a new node with the type of CallExpression. This node will have two properties: name - the name of the function we are calling; and params - the function's arguments. Our current token is the paren token, which doesn't hold the function's name. Since we know how our tokenizer works and that the next token after an opening parenthesis holds the name of the function, we'll advance to the next token and take the name from there. Don't forget to change token to a let variable.
- We'll now want to begin collecting our params. Since we know that the next token after the name token is the first param, we'll advance to the next token. We'll now iterate the tokens until we reach the end of the function call - the closing parenthesis sign ). We will call our walk function and add it's return value to the params array. We take into account that the walk function increments the current pointer, so we need to assign the current token without incrementing the counter (line 22). When we find the closing parenthesis token, we exit the loop, increment our counter so the next walk iteration will handle the next token (and not the ) token), and return our expression.
- A nice thing to note here is that since function calls can be nested (like our example - (add 2 (sub 4 3))), the second param of the add function will be sub function CallExpression.
- If we follow our tokens. This is how the program will execute.

1. {type: 'paren', value: '('}
2. {type: 'name', value: 'add'}
3. {type: 'number', value: '2'}
4. {type: 'paren', value: '('}
5. {type: 'name', value: 'sub'}
6. {type: 'number', value: '4'}
7. {type: 'number', value: '3'}
8. {type: 'paren', value: ')'}
9. {type: 'paren', value: ')'}
If we run our program we should see our beautiful tree.

# Transformation

- We now have an easy-to-work-with representation of our original code (Lisp). We now want to convert it to an easy-to-work-with representation of the code we want to create (Javascript).
- We'll create a transformer function that receives the AST we just created and returns a new Javascript AST.
- We'll create a new transfomer.js file and define our function. As in the Syntactic Analysis phase, we'll create an AST object with a root node of Program.
Remember, we want to transform one tree into another. For that, we'll use a function called traverse. This function doesn't exist yet, and we'll implement it later.

- The traverse function receives an AST and an object that we'll expand upon in the next step.
- Our AST is comprised of different nodes where each node has a type. The traverse function will start "traveling" upon our tree, and for each node that it will encounter, it will search for a corresponding function in the object we passed it (the 2nd argument). Don't worry if it's a bit confusing right now. It will be much clearer once we implement it.
- An easy place to start is with our "leaf" nodes - the nodes we know that don't have any children. In our case, it's NumberLiteral. We'll define a NumberLiteral method. This method receives the node as an argument. This method's role is to handle the node accordingly, meaning - to add it to the new AST in a correct fashion.

- The question is, how do we know where to add it in the new tree?
Let's use a non-existent "magic" variable to point to the correct place in the new AST where we should add any new nodes. We'll handle this magic variable later. We'll push a new node to that magic pointer. Our new node will be of type NumericLiteral, which is how we represent numbers in our new AST.
We'll now want to handle the function calling node - CallExpression. We'll create an appropriate method and create a new expression to represent our new node.
In our original AST, the CallExpression node had a name property that held the function's name and a params property that held its arguments. In our new AST, the name of the called function will be in a sub-object called callee, and we'll change the params property to arguments.
We can now add our new expression under the position pointer. Afterward, we'll change the value of position to our arguments array since we want to add any new nodes under this AST branch.
- Let's complicate it a bit. In our new AST, any function call will be of type CallExpression, but the top-most function call will be of type ExpressionStatement. - For that reason, we need to check if the current node's parent is not also a CallExpression, then we know we're a top-level function call. We'll wrap the existing expression in an ExpressionStatement node (lines 27-30).
For that, we'll need to receive the parent as a parameter in our CallExpression function. Remember, we haven't even implemented the traverse function yet, so we can feel free to add functionality as we see fit.
- One last thing left in the transformer function is to take care of our "magic" variable position. We'll define our position variable and point it at the body property of our root node - since this is the place we want to start our new tree.
- We're done with the transformer function, but if we run our program, it will explode! We haven't implemented the traverse function yet! Let's do that next.


# Traverse

- In the last step, we traversed our original AST and built a new AST. Now let's implement the traverse function.
- We'll create a new file called traverse.js and export a function that accepts an AST and an object. If you remember from the last step, this object holds the methods we want to execute for each node according to its type. This pattern is called a visitor pattern - and we'll name our object visitors accordingly.
- We'll create a function called walkNode that will be called on each node in the tree. We'll check if our visitors object contains a method corresponding to the node type and call it if it exists.
- We'll now handle some special cases. One of them is our AST's root - the Program node, which has a body property. Since the body property is of type array, we'll want to iterate it, and for each member, we'll call the walkNode separately.
- A second special case is the CallExpression node, which has the params array property. We'll handle it the same way.
- The last thing left is to kick off the first "walk" by calling walkNode on the root of the AST.
Since we completed the transformation phase, we can run our program and see our new AST.
# Code Generator

- The last step in our (simple) compiler is actually to generate the Javascript code.
- We'll create a generateCode function and file and call it with our jsAST.
- Since our code is a string, we can drop the call to JSON.stringify on our output.
- We'll create our last file - generateCode.js and define our function. This function will recursively build a string output according to our AST. Let's handle the easy nodes first.
- If we run into a NumericLiteral node, we'll just print out the value.
- If we run into an Identifier node (the function's name, in our simple case), we'll print out the name.
- When handling the CallExpression (the function call) we need to retrieve both the name of the function (callee) and the arguments. Since the callee property is an Identifier which we already handle in a different condition, we can pass it back to the generateCode function and expect to receive the name back. To handle the arguments we can map the arguments property and call generateCode on each of them. We'll receive an array back containing the different string representations of the arguments. We can join the array with commas, the argument separator in Javascript.
- The ExpressionStatement, our top-level function call, will call generateCode on its expression parameter and add a semicolon at the end.
- The last step will handle our root node - Program. We'll iterate the body property and output a different line of code for each member (which will only be one in our simple case).