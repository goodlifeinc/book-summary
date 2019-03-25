## Chapter 2: Meaningful names
1. **Choose names that reveal intent** (choose them thoughtfully) - they are a tool that you can use to communicate with others. If you have to put a comment that describes your name, it is a bad name.
    
2. **Avoid disinformation** - avoid words whose entrenched meanings vary from our intended meaning. Do not add number series or noise words. If names must be different, then they should also mean something different. Also the word variable should never appear in a variable name (The word table should never appear in a table name).
    - Even if you are coding a hypotenuse and ```hp``` looks like a good abbreviation, it could be disinformative.
    - Do not refer to a grouping of accounts as an ```accountList``` unless it’s actually a ```List```(instead use: ```accountGroup```, ```bunchOfAccounts```, etc.)
    - Beware of using names which vary in small ways. Takes a long time to spot the difference:
        ```XYZControllerForEfficientHandlingOfStrings```
        ```XYZControllerForEfficientStorageOfStrings```
    - Number-series naming (```a1```, ```a2```, ```aN```) are not disinformative — they are noninformative. They provide no clue to the author’s intention.
    - Noise words are another meaningless distinction - If you have a ```Product``` class and another called ```ProductInfo``` or ```ProductData```, you have made the names different without making them mean anything different. Info and Data are indistinct noise words like a, an, and the.
   - Never use ```NameString``` instead of just ```Name```.
        
    ***Remember:*** If you have to read the code to understand what it means, that is a bad name.

3. **Use Pronounceable Names** - other people will have to talk about your code, so make it easy for them. Pick names that they can say. Do not use something like ```genymdhms```, ```getYYYY```, ect.

4. **Use Searchable Names** - single-letter names and numeric constants are not easy to locate across a body of text. Single-letter names can ONLY be used as local variables inside short methods. *The length of a name should correspond to the size of its scope*

5. **Avoid Encodings** - simply adds an extra burden. Do not put ```I``` infront interfaces. Instead of naming it ```IShapeFactory```, use ```ShapeFactory```. You don’t need to prefix member variables with ```m_``` ot anything similar. Your classes and functions should be small enough that you don’t need them. Encodings make your code ***HARD TO READ.***

6. **Choose parse of speech well** 
   - Class names - classes and objects should have noun or noun phrase. A class name should not be a verb.
Use ```Customer```, ```WikiPage```, ```Account```, and ```AddressParser```and avoid: ```Manager```, ```Processor```, ```Data```, ```Info```.
   - Method names - methods should have verb or verb phrase names. Accessors, mutators, and predicates should be named for their value and prefixed with ```get```, ```set```, and ```is```. Use ```postPayment```, ```deletePage```, ```save```, ```getName```, ```setName```, ```isPosted```.
   - Pick One Word per Concept - it’s confusing to have a controller and a manager and a driver in the same code base.
   - Avoid using the same word for two purposes - instead of using only add for everything, use insert or append instead.

7. **The scope rule** - variable names should be short, even one letter if they are in a tiny little scope, but they should be long if they are in a big, long scope. Global variable names should be long. Functions follows the oposite rule: Function names should be short, if they've got a long scope, and they should be long and descriptive if they have a short scope.The same rule goes for classes: short names for public classes and longer names for private classes in tiny scopes.
