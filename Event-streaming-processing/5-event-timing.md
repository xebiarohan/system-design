## Event timings

1. There are multiple timestamps associted with an event
    Event time : Time when the event happen
    Arrival time : Time when the event is consumed by the application
    Processing time: Time when the event is processed by the consumer

2. Based on the requirement we can either use event time or the processing time for event stream processing

3. We use event time for processes like
    - Stock prices
    - GPS coordinates


4. We use Arrival/Processing time
    - time limiting algorithm
    - User actions analysis


5. Handling late events
    - TODO
