## Chapter 2: Meaningful names
Rules:
1. **Choose names that reveal intent**
    *Ex:*
    ``` java
   public List<Cell> getFlaggedCells() {
        List<Cell> flaggedCells = new ArrayList<Cell>();
        for (Cell cell : gameBoard){
            if (cell.isFlagged()){
                flaggedCells.add(cell);   
            }
        }
        
        return flaggedCells;        
    }
    ```
    
2. **Avoid disinformation** - avoid words whose entrenched meanings vary from our intended meaning
    *Ex:*
    - Even if you are coding a hypotenuse and ```hp``` looks like a good abbreviation, it could be disinformative.
    - Do not refer to a grouping of accounts as an ```accountList``` unless it’s actually a ```List```(instead use: ```accountGroup```, ```bunchOfAccounts```, etc.)
    - Beware of using names which vary in small ways. Takes a long time to spot the difference:
        ```XYZControllerForEfficientHandlingOfStrings```
        ```XYZControllerForEfficientStorageOfStrings```

3. **Make Meaningful Distinctions** - It is not sufficient to add number series or noise words. If names must be different, then they should also mean something different.
    - Number-series naming (```a1```, ```a2```, .. ```aN```) are not disinformative—they are noninformative. They provide no clue to the author’s intention.
    - Noise words are another meaningless distinction:
        *Ex:* If you have a ```Product``` class and another called ```ProductInfo``` or ```ProductData```, you have made the names different without making them mean anything different. Info and Data are indistinct noise words like a, an, and the.

4. **Noise words are redundant**. The word variable should never appear in a variable
name. The word table should never appear in a table name. 
    *Ex:* You should never use ```NameString``` instead of just ```Name```.

5. **Use Pronounceable Names**
    *Ex:*
    ```genymdhms```????

6. **Use Searchable Names**
    *Ex:* Single-letter names and numeric constants are not easy to locate across a body of text. Single-letter names can ONLY be used as local variables inside short methods. *The length of a name should correspond to the size of its scope*

7. **Avoid Encodings** - simply adds an extra burden

8. **Member Prefixes** - you don’t need to prefix member variables with ```m_```. Your classes and functions should be small enough that you don’t need them.

9. **Interfaces and Implementations**
    *Ex:* If you hava an interface instead of naming it ```IShapeFactory```, use ```ShapeFactory```. You don’t want users to know that you are handing them an interface. You just want them to know that it’s a ```ShapeFactory```. Implementation name could be ```ShapeFactoryImp```.

10. **Avoid Mental Mapping**

11. **Class names** - classes and objects should have noun or noun phrase. A class name should not be a verb.
*Ex:* ```Customer```, ```WikiPage```, ```Account```, and ```AddressParser```. 
    Avoid: ```Manager```, ```Processor```, Data```, ```Info```

12. **Method names** - methods should have verb or verb phrase names. Accessors, mutators, and predicates should be named for their value and prefixed with ```get```, ```set```, and ```is```.
    *Ex:* ```postPayment```, ```deletePage```, ```save```, ```getName```, ```setName```, ```isPosted```.

13. **Pick One Word per Concept** - it’s confusing to have a controller and a manager and a driver in the same code base.

14. **Don’t Pun** - Avoid using the same word for two purposes.

15. **Use Solution Domain Names** - use computer science (CS) terms, algorithm names, pattern names, math terms, and so forth.

16. **Use Problem Domain Names**

17. **Add Meaningful Context**

18. **Don’t Add Gratuitous Context**

## Chapter 3 : Functions

## Chapter 4 : Comments

## Chapter 5 : Formatting

## Chapter 6 : Objects and data structures

## Chapter 7 : Error handling

## Chapter 8 : Boundaries

## Chapter 9 : Unit tests

## Chapter 10 : Classes

## Chapter 11 : Systems
