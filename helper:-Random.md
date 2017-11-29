### `VariableAmount`
A VariableAmount represents a random object which returns a possibly varying value on each execution of #get.

```java
// randomly returns a value between 5 and 9
VariableAmount amount = VariableAmount.range(5, 9);

// returns a different value each time
int amt1 = amount.getFlooredAmount();
int amt2 = amount.getFlooredAmount();
int amt3 = amount.getFlooredAmount();

// returns a base value of 10, with a random amount between 0 and 5 added/subtracted
VariableAmount amount2 = VariableAmount.baseWithVariance(10, 5);

// returns a base value of 50, with a 50% chance of also applying a varience +/- between 0 and 10
VariableAmount amount3 = VariableAmount.baseWithOptionalVariance(50, 10, 0.5);
```

### `RandomSelector`
RandomSelector is a tool which will randomly select from a collection of objects.

```java
RandomSelector<String> stringSelector = RandomSelector.uniform(ImmutableList.of("string1", "string2", "string3"));

// has an equal chance of selecting each of the 3 strings
String picked = stringSelector.pick();
```

The selector also supports weighted values - where each element in the collection is assigned a weight. Elements with a higher weight are more likely to be selected.

```java
// define the elements to pick from
enum Prize implements Weighted {
    COMMON(5.0),
    RARE(0.5),
    SUPER_RARE(0.1);

    private final double weight;

    Prize(double weight) {
        this.weight = weight;
    }

    @Override
    public double getWeight() {
        return weight;
    }
}

// make a weighted selector
RandomSelector<Prize> selector = RandomSelector.weighted(Arrays.asList(Prize.values()));

Prize prize1 = selector.pick();
Prize prize2 = selector.pick();
Prize prize3 = selector.pick();
```

