## Hopping window

1. Features
    - It is similar to tumbling window in the sense that it also has fixed window sizes
    - It also return the processed result only after the window time is finished
    - The main difference is in hopping window the windows are overlapping
    - As the windows are overlapping, the data we receive it quicker than tumbling window


```text
        -----------
    -----------
---------
```

2. Use cases
   - Stock trading -prices of stocks
   - Error logs analytics

3. Pros
   - Produce results more frequently
   - Great for UIs managed by operators

4. Cons
   - Aggregating all the data in memory
   - Need to maintain multiple windows now (instead of 1 at a time in Tumbling window)
   - Duplicate events stores in memory