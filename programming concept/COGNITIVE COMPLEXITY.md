This metric indicates how much it's difficult for a human to **understand** the code and all its possible paths. **Cognitive complexity** will give more weight to nested conditions as it's supposed to be harder to read. If we consider our previous example, we get a cognitive complexity of 6.

```c#
List<Integer> findCommonNumbers(List<Integer> sourceList, List<Integer> candidateList) {           
    List<Integer> result = new ArrayList<>();
    for (int i = 0 ; i < sourceList.size() ; i++) {    //+1
        for (int j = 0 ; j < candidateList.size() ; j++) { //+2 (nested level = 2)
            if (sourceList.get(i) == candidateList.get(j)) {  //+3 (nested level = 3)            
                    result.add(source1.get(i)); 
            }  
    }
    return result;
}
```

There is also a major difference in some structures like **switch/case**. In this example :
```C#
int getCountryTax(String country) {
    switch (country) {          
      case "FR":
        return 20;
      case "EN":
        return 10;
      case "US":
        return 5;
      default:
        return 0;
    }
}
```
We'll get a Cyclomatic complexity of 4, while we have a Cognitive Complexity of 1 only. This second metric estimates that the **code is not harder to understand** if we have 4 or 7 cases in our switch.

You can find more details on this metric [here](https://www.sonarsource.com/docs/CognitiveComplexity.pdf), along with its motivations and the issues it tries to resolve.