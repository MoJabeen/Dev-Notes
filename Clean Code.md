Description: Summary of notes from Clean Code by Robert Cecil Martin

Each section is generally broken into a list of requirements 

# Clean Code

The total cost of owning a messy code base (or any foundation) will drive productivity to 0.

Weak foundations also progressively increase the difficulty of effective improvements.

~ Good engineering enables future iterative improvement and effective scaling.

## Variable Names

- Names should show their intention
- Avoid multiple names with small changes between them
- Avoid names with other known meanings
- Avoid meaningless names like data, var etc
- Check if names are pronounceable
- The length of the name should correspond to the scope of the var
  
- Test good naming scheme has no mental mapping of names
- Method names should be verbs
- Use known subject domain names
- Create names that give context : Classes are great way to link context

## Functions

- Keep functions very small, should be < 20 lines long
- Should **do only one thing** (the name of the function)
- Condition statements (if, else, while) should only be one line long; call a function
- Keep to one level of abstraction in the function

- Switch statements should be used in a low level class creating new instances instead of jumping to functions
  - Use polymorphism to determine the function based on the object
  - Use design type : ABSTRACT FACTORY

- Long names for functions are accepted

- Minimise number of arguments (use structs and objects)

- There are three forms of functions:
  - Questions, getting info about the arg
  - Operative, transform arg and return it
  - Event, Trigger change of state based on arg

- Avoid flag arguments
- A list of related variables can be considered one arg

    meh(int blah, int ...args)

- Function arguments should not be changed in the function (the output should be via return)
- Remove any duplication of calling functions !!

## Comments

- Good comments dont replace bad code !!
  - Use good code instead of good comments

- Good for showing intent (why you made this decision)
- Use comments to add extra clarification on confusing syntax
- Show warnings on important areas of code
- Can be used with IDEs for Todos

- Avoid statements that are not completely accurate
- Avoid commented banners instead use more effectively organized code.

- DONT COMMENT OUT CODE, delete it; rely on source control to retrieve old code.

## Formatting

- Small files are preferred
- Related concepts should not be in different files
  - Test: Avoid needing to jump between many files to understand a file

- Instance variables are placed at the top
- Caller functions should be placed above the callee
- Dependencies should flow down through the code
- Avoid very wide lines of code