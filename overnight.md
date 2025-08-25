```mermaid
%%{init: {'theme': 'neutral'}}%%
flowchart TB
    subgraph Base["Overnight Risk Grid"]
        direction TB

        BasePV["Scenario: Base
        AAPL=200, MSFT=100
        AAPL vol=25%, MSFT vol=20%
        Portfolio PV = 10,000,000"]

        AAPLup["Scenario: AAPL +1
        AAPL=201, MSFT=100
        PV = 10,100,000"]

        AAPLdown["Scenario: AAPL –1
        AAPL=199, MSFT=100
        PV = 9,900,000"]

        MSFTup["Scenario: MSFT +1
        AAPL=200, MSFT=101
        PV = 9,950,000"]

        MSFTdown["Scenario: MSFT –1
        AAPL=200, MSFT=99
        PV = 10,050,000"]

        AAPLvolup["Scenario: AAPL Vol +1%
        AAPL=200, MSFT=100
        AAPL vol=26%
        PV = 10,030,000"]

        AAPLvoldown["Scenario: AAPL Vol –1%
        AAPL=200, MSFT=100
        AAPL vol=24%
        PV = 9,970,000"]
    end

    BasePV --> AAPLup
    BasePV --> AAPLdown
    BasePV --> MSFTup
    BasePV --> MSFTdown
    BasePV --> AAPLvolup
    BasePV --> AAPLvoldown

```
