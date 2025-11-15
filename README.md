# ATM Machine System - Design Documentation

## Requirements

### Functional Requirements
1. **Card Authentication**: Insert card and validate PIN
2. **Transaction Selection**: Choose between cash withdrawal and balance check
3. **Cash Withdrawal**: Withdraw amount with optimal note distribution
4. **Balance Inquiry**: Check account balance
5. **ATM Balance Management**: Track ATM cash inventory (₹2000, ₹500, ₹100 notes)
6. **User Account Management**: Maintain user bank account balance
7. **State Transitions**: Machine transitions through appropriate states during transaction

### Non-Functional Requirements
1. **Security**: PIN validation before any transaction
2. **Reliability**: Handle insufficient funds (ATM and user account)
3. **Maintainability**: Easy to add new transaction types or note denominations
4. **Singleton Pattern**: Single ATM instance throughout application

## Objectives

### Primary Objectives
1. **Implement State Pattern**: Manage ATM behavior across different states
2. **Implement Chain of Responsibility**: Handle cash withdrawal with multiple note processors
3. **Separation of Concerns**: Isolate authentication, transaction, and cash dispensing logic
4. **Error Handling**: Gracefully handle insufficient funds and invalid operations

### Design Objectives
1. Create a flexible architecture supporting future enhancements
2. Minimize coupling between components
3. Follow SOLID principles
4. Use appropriate design patterns for each responsibility

## Design Patterns Used

### 1. State Pattern (Implemented)
**Purpose**: Manage ATM behavior that changes based on internal state

**Implementation**:
- **Context**: `ATM` class maintains current state
- **State Interface**: Abstract `ATMState` class defines all possible operations
- **Concrete States**: 
  - `IdleState`: Waiting for card insertion
  - `HasCardState`: Card inserted, waiting for PIN
  - `SelectOperationState`: PIN validated, selecting transaction type
  - `CashWithdrawalState`: Processing cash withdrawal
  - `CheckBalanceState`: Displaying balance

**Benefits**:
- Eliminates complex conditional logic
- Each state encapsulates its own behavior
- Easy to add new states (e.g., CardTransferState, DepositState)
- State transitions are explicit and controlled

**State Transitions**:
```
IdleState → HasCardState (insert card)
HasCardState → SelectOperationState (correct PIN)
HasCardState → IdleState (wrong PIN)
SelectOperationState → CashWithdrawalState (withdraw)
SelectOperationState → CheckBalanceState (check balance)
CashWithdrawalState → IdleState (transaction complete)
CheckBalanceState → IdleState (transaction complete)
```

### 2. Chain of Responsibility Pattern (Implemented)
**Purpose**: Handle cash withdrawal by processing note denominations in sequence

**Implementation**:
- **Handler**: Abstract `CashWithdrawProcessor` class
- **Concrete Handlers**: 
  - `TwoThousandWithdrawProcessor`: Processes ₹2000 notes first
  - `FiveHundredWithdrawProcessor`: Processes ₹500 notes second
  - `OneHundredWithdrawProcessor`: Processes ₹100 notes last

**How it works**:
```java
CashWithdrawProcessor withdrawProcessor = 
    new TwoThousandWithdrawProcessor(
        new FiveHundredWithdrawProcessor(
            new OneHundredWithdrawProcessor(null)
        )
    );

withdrawProcessor.withdraw(atmObject, 2700);
// Chain processes: 2000 → 500 → 100
// Result: 1x₹2000 + 1x₹500 + 2x₹100
```

**Benefits**:
- Decouples sender from receiver
- Easy to add new note denominations (e.g., ₹200, ₹50)
- Each processor has single responsibility
- Flexible chain composition

### 3. Singleton Pattern (Implemented)
**Purpose**: Ensure single ATM instance

**Implementation**:
```java
public class ATM {
    private static ATM atmObject = new ATM(); // Eager initialization
    
    private ATM() {}
    
    public static ATM getATMObject() {
        atmObject.setCurrentATMState(new IdleState());
        return atmObject;
    }
}
```

**Benefits**:
- Single point of control for ATM resources
- Consistent state across application
- Thread-safe with eager initialization

## UML Class Diagram

```mermaid
classDiagram
    %% ========================================
    %% CORE ATM CONTEXT (Singleton)
    %% ========================================
    class ATM {
        -ATM atmObject$
        -ATMState currentATMState
        -int noOfTwoThousandNotes
        -int noOfFiveHundredNotes
        -int noOfOneHundredNotes
        -int atmBalance
        -ATM()
        +getATMObject()$ ATM
        +getCurrentATMState() ATMState
        +setCurrentATMState(ATMState) void
        +getAtmBalance() int
        +setAtmBalance(int, int, int, int) void
        +getNoOfTwoThousandNotes() int
        +getNoOfFiveHundredNotes() int
        +getNoOfOneHundredNotes() int
        +deductATMBalance(int) void
        +deductTwoThousandNotes(int) void
        +deductFiveHundredNotes(int) void
        +deductOneHundredNotes(int) void
        +printCurrentATMStatus() void
    }

    %% ========================================
    %% STATE PATTERN - Abstract State
    %% ========================================
    class ATMState {
        <<abstract>>
        +insertCard(ATM, Card) void
        +authenticatePin(ATM, Card, int) void
        +selectOperation(ATM, Card, TransactionType) void
        +cashWithdrawal(ATM, Card, int) void
        +displayBalance(ATM, Card) void
        +returnCard() void
        +exit(ATM) void
    }

    %% ========================================
    %% STATE PATTERN - Concrete States
    %% ========================================
    class IdleState {
        +insertCard(ATM, Card) void
    }

    class HasCardState {
        +HasCardState()
        +authenticatePin(ATM, Card, int) void
        +exit(ATM) void
        +returnCard() void
    }

    class SelectOperationState {
        +SelectOperationState()
        +selectOperation(ATM, Card, TransactionType) void
        +exit(ATM) void
        +returnCard() void
        -showOperations() void
    }

    class CashWithdrawalState {
        +CashWithdrawalState()
        +cashWithdrawal(ATM, Card, int) void
        +exit(ATM) void
        +returnCard() void
    }

    class CheckBalanceState {
        +CheckBalanceState()
        +displayBalance(ATM, Card) void
        +exit(ATM) void
        +returnCard() void
    }

    %% ========================================
    %% CHAIN OF RESPONSIBILITY - Abstract Handler
    %% ========================================
    class CashWithdrawProcessor {
        <<abstract>>
        #CashWithdrawProcessor nextCashWithdrawalProcessor
        +CashWithdrawProcessor(CashWithdrawProcessor)
        +withdraw(ATM, int) void
    }

    %% ========================================
    %% CHAIN OF RESPONSIBILITY - Concrete Handlers
    %% ========================================
    class TwoThousandWithdrawProcessor {
        +TwoThousandWithdrawProcessor(CashWithdrawProcessor)
        +withdraw(ATM, int) void
    }

    class FiveHundredWithdrawProcessor {
        +FiveHundredWithdrawProcessor(CashWithdrawProcessor)
        +withdraw(ATM, int) void
    }

    class OneHundredWithdrawProcessor {
        +OneHundredWithdrawProcessor(CashWithdrawProcessor)
        +withdraw(ATM, int) void
    }

    %% ========================================
    %% USER & ACCOUNT COMPONENTS
    %% ========================================
    class User {
        -Card card
        -UserBankAccount bankAccount
        +getCard() Card
        +setCard(Card) void
    }

    class Card {
        -int PIN_NUMBER$
        -int cardNumber
        -int cvv
        -int expiryDate
        -int holderName
        -UserBankAccount bankAccount
        +isCorrectPINEntered(int) boolean
        +getBankBalance() int
        +deductBankBalance(int) void
        +setBankAccount(UserBankAccount) void
    }

    class UserBankAccount {
        -int balance
        +withdrawalBalance(int) void
        +getBalance() int
        +setBalance(int) void
    }

    %% ========================================
    %% ENUMERATIONS
    %% ========================================
    class TransactionType {
        <<enumeration>>
        CASH_WITHDRAWAL
        BALANCE_CHECK
        +showAllTransactionTypes()$ void
    }

    %% ========================================
    %% MAIN APPLICATION
    %% ========================================
    class ATMRoom {
        -ATM atm
        -User user
        +main(String[])$ void
        -initialize() void
        -createUser() User
        -createCard() Card
        -createBankAccount() UserBankAccount
    }

    %% ========================================
    %% STATE PATTERN RELATIONSHIPS
    %% ========================================
    ATMState <|-- IdleState : extends
    ATMState <|-- HasCardState : extends
    ATMState <|-- SelectOperationState : extends
    ATMState <|-- CashWithdrawalState : extends
    ATMState <|-- CheckBalanceState : extends
    
    ATM *-- ATMState : contains
    
    %% ========================================
    %% CHAIN OF RESPONSIBILITY RELATIONSHIPS
    %% ========================================
    CashWithdrawProcessor <|-- TwoThousandWithdrawProcessor : extends
    CashWithdrawProcessor <|-- FiveHundredWithdrawProcessor : extends
    CashWithdrawProcessor <|-- OneHundredWithdrawProcessor : extends
    
    CashWithdrawProcessor o-- CashWithdrawProcessor : next
    
    %% ========================================
    %% STATE USES CHAIN
    %% ========================================
    CashWithdrawalState ..> CashWithdrawProcessor : uses
    CashWithdrawalState ..> TwoThousandWithdrawProcessor : creates
    CashWithdrawalState ..> FiveHundredWithdrawProcessor : creates
    CashWithdrawalState ..> OneHundredWithdrawProcessor : creates
    
    %% ========================================
    %% USER & ACCOUNT RELATIONSHIPS
    %% ========================================
    User *-- Card : has
    User *-- UserBankAccount : has
    Card --> UserBankAccount : references
    
    %% ========================================
    %% STATE TRANSITIONS
    %% ========================================
    IdleState ..> HasCardState : creates
    HasCardState ..> SelectOperationState : creates
    HasCardState ..> IdleState : creates
    SelectOperationState ..> CashWithdrawalState : creates
    SelectOperationState ..> CheckBalanceState : creates
    SelectOperationState ..> IdleState : creates
    CashWithdrawalState ..> IdleState : creates
    CheckBalanceState ..> IdleState : creates
    
    %% ========================================
    %% OTHER RELATIONSHIPS
    %% ========================================
    ATMRoom --> ATM : uses
    ATMRoom --> User : uses
    SelectOperationState ..> TransactionType : uses
    
    ATMState ..> Card : uses
    ATMState ..> TransactionType : uses
```

## State Transition Diagram

```mermaid
stateDiagram-v2
    [*] --> IdleState: ATM Initialized
    
    IdleState --> HasCardState: insertCard()
    
    HasCardState --> SelectOperationState: authenticatePin() [Valid PIN]
    HasCardState --> IdleState: authenticatePin() [Invalid PIN] / Return Card
    HasCardState --> IdleState: exit() / Return Card
    
    SelectOperationState --> CashWithdrawalState: selectOperation(CASH_WITHDRAWAL)
    SelectOperationState --> CheckBalanceState: selectOperation(BALANCE_CHECK)
    SelectOperationState --> IdleState: exit() / Return Card
    
    CashWithdrawalState --> IdleState: cashWithdrawal() [Complete] / Return Card
    CheckBalanceState --> IdleState: displayBalance() [Complete] / Return Card
    
    note right of IdleState
        Waiting for card
        ATM ready for transaction
    end note
    
    note right of HasCardState
        Card inserted
        Awaiting PIN validation
    end note
    
    note right of SelectOperationState
        PIN verified
        Choose transaction type
    end note
    
    note right of CashWithdrawalState
        Process withdrawal
        Use Chain of Responsibility
        for note distribution
    end note
    
    note right of CheckBalanceState
        Display balance
        Read-only operation
    end note
```

## Chain of Responsibility Flow Diagram

```mermaid
flowchart TD
    Start([Cash Withdrawal Request<br/>Amount: ₹2700]) --> TwoK[TwoThousandWithdrawProcessor]
    
    TwoK --> TwoKCalc{Calculate:<br/>2700 ÷ 2000 = 1 note<br/>Remaining: ₹700}
    TwoKCalc --> TwoKDispense[Dispense: 1 × ₹2000]
    TwoKDispense --> TwoKDeduct[ATM deducts 1 note]
    TwoKDeduct --> TwoKNext{Balance > 0?}
    TwoKNext -->|Yes: ₹700| FiveH[FiveHundredWithdrawProcessor]
    
    FiveH --> FiveHCalc{Calculate:<br/>700 ÷ 500 = 1 note<br/>Remaining: ₹200}
    FiveHCalc --> FiveHDispense[Dispense: 1 × ₹500]
    FiveHDispense --> FiveHDeduct[ATM deducts 1 note]
    FiveHDeduct --> FiveHNext{Balance > 0?}
    FiveHNext -->|Yes: ₹200| OneH[OneHundredWithdrawProcessor]
    
    OneH --> OneHCalc{Calculate:<br/>200 ÷ 100 = 2 notes<br/>Remaining: ₹0}
    OneHCalc --> OneHDispense[Dispense: 2 × ₹100]
    OneHDispense --> OneHDeduct[ATM deducts 2 notes]
    OneHDeduct --> OneHNext{Balance > 0?}
    OneHNext -->|No: ₹0| Complete([Transaction Complete<br/>Total: 1×₹2000 + 1×₹500 + 2×₹100])
    
    TwoKNext -->|No| Complete
    FiveHNext -->|No| Complete
    
    style Start fill:#e3f2fd
    style TwoK fill:#fff3e0
    style FiveH fill:#fff3e0
    style OneH fill:#fff3e0
    style Complete fill:#c8e6c9
    style TwoKDispense fill:#ffe0b2
    style FiveHDispense fill:#ffe0b2
    style OneHDispense fill:#ffe0b2
```

## Sequence Diagram - Complete Transaction Flow

```mermaid
sequenceDiagram
    participant User
    participant ATMRoom
    participant ATM
    participant IS as IdleState
    participant HCS as HasCardState
    participant SOS as SelectOperationState
    participant CWS as CashWithdrawalState
    participant TwoK as TwoThousandProcessor
    participant FiveH as FiveHundredProcessor
    participant OneH as OneHundredProcessor
    participant Card
    participant Bank as UserBankAccount

    Note over ATMRoom: System Initialization
    ATMRoom->>ATM: getATMObject()
    activate ATM
    ATM->>IS: new IdleState()
    ATM-->>ATMRoom: ATM instance
    deactivate ATM
    ATMRoom->>ATM: setAtmBalance(3500, 1, 2, 5)
    Note over ATM: ATM Balance: ₹3500<br/>1×₹2000, 2×₹500, 5×₹100

    Note over User,ATMRoom: Card Insertion Phase
    User->>ATMRoom: Insert Card
    ATMRoom->>ATM: insertCard(card)
    ATM->>IS: insertCard(card)
    activate IS
    IS->>ATM: setCurrentATMState(HasCardState)
    deactivate IS
    Note over ATM: State: HasCardState

    Note over User,Card: PIN Authentication Phase
    User->>ATMRoom: Enter PIN: 112211
    ATMRoom->>ATM: authenticatePin(card, 112211)
    ATM->>HCS: authenticatePin(card, 112211)
    activate HCS
    HCS->>Card: isCorrectPINEntered(112211)
    activate Card
    Card-->>HCS: true
    deactivate Card
    HCS->>ATM: setCurrentATMState(SelectOperationState)
    deactivate HCS
    Note over ATM: State: SelectOperationState

    Note over User,SOS: Operation Selection Phase
    User->>ATMRoom: Select CASH_WITHDRAWAL
    ATMRoom->>ATM: selectOperation(CASH_WITHDRAWAL)
    ATM->>SOS: selectOperation(CASH_WITHDRAWAL)
    activate SOS
    SOS->>ATM: setCurrentATMState(CashWithdrawalState)
    deactivate SOS
    Note over ATM: State: CashWithdrawalState

    Note over User,Bank: Cash Withdrawal Phase
    User->>ATMRoom: Withdraw ₹2700
    ATMRoom->>ATM: cashWithdrawal(card, 2700)
    ATM->>CWS: cashWithdrawal(card, 2700)
    activate CWS
    
    CWS->>ATM: getAtmBalance()
    ATM-->>CWS: 3500 (sufficient)
    CWS->>Card: getBankBalance()
    Card-->>CWS: 3000 (sufficient)
    
    CWS->>Card: deductBankBalance(2700)
    Card->>Bank: withdrawalBalance(2700)
    Note over Bank: User Balance: ₹300
    
    CWS->>ATM: deductATMBalance(2700)
    Note over ATM: ATM Balance: ₹800

    Note over CWS,OneH: Chain of Responsibility Execution
    CWS->>TwoK: new TwoThousandProcessor(FiveHundred(OneHundred(null)))
    CWS->>TwoK: withdraw(atm, 2700)
    activate TwoK
    TwoK->>TwoK: Calculate: 2700 ÷ 2000 = 1
    TwoK->>ATM: deductTwoThousandNotes(1)
    Note over ATM: Notes: 0×₹2000
    TwoK->>FiveH: withdraw(atm, 700)
    activate FiveH
    FiveH->>FiveH: Calculate: 700 ÷ 500 = 1
    FiveH->>ATM: deductFiveHundredNotes(1)
    Note over ATM: Notes: 1×₹500
    FiveH->>OneH: withdraw(atm, 200)
    activate OneH
    OneH->>OneH: Calculate: 200 ÷ 100 = 2
    OneH->>ATM: deductOneHundredNotes(2)
    Note over ATM: Notes: 3×₹100
    OneH-->>FiveH: Complete
    deactivate OneH
    FiveH-->>TwoK: Complete
    deactivate FiveH
    TwoK-->>CWS: Complete
    deactivate TwoK

    Note over CWS,ATM: Transaction Completion
    CWS->>CWS: exit()
    CWS->>CWS: returnCard()
    CWS->>ATM: setCurrentATMState(IdleState)
    deactivate CWS
    Note over ATM: State: IdleState

    Note over User: Transaction Complete!<br/>Dispensed: 1×₹2000 + 1×₹500 + 2×₹100
    ATMRoom->>ATM: printCurrentATMStatus()
    Note over ATM: Final State:<br/>Balance: ₹800<br/>0×₹2000, 1×₹500, 3×₹100
```

## Class Responsibilities

### Core ATM Components

#### ATM (Singleton Context)
- **Purpose**: Central coordinator for ATM operations
- **Responsibilities**:
  - Maintain current state
  - Manage cash inventory (₹2000, ₹500, ₹100 notes)
  - Track total ATM balance
  - Provide single instance access
  - Deduct notes and balance on withdrawal
- **Design Pattern**: Singleton (eager initialization)
- **Key Methods**: 
  - `getATMObject()`: Returns singleton instance
  - `setCurrentATMState()`: Changes state
  - `deductTwoThousandNotes()`, `deductFiveHundredNotes()`, `deductOneHundredNotes()`: Update inventory

### State Pattern Components

#### ATMState (Abstract)
- **Purpose**: Define interface for all ATM operations
- **Responsibilities**:
  - Declare all state-specific operations
  - Provide default error implementations
  - Allow subclasses to override relevant methods
- **Pattern Role**: State interface in State Pattern
- **Default Behavior**: Print "OOPS!! Something went wrong" for invalid operations

#### IdleState
- **Purpose**: Initial waiting state
- **Responsibilities**:
  - Accept card insertion
  - Transition to HasCardState
- **Valid Operations**: `insertCard()`
- **Entry Message**: None (silent state)

#### HasCardState
- **Purpose**: Card inserted, awaiting authentication
- **Responsibilities**:
  - Display "enter your card pin number"
  - Validate PIN against Card
  - Transition to SelectOperationState (success)
  - Return to IdleState (failure)
  - Return card on exit
- **Valid Operations**: `authenticatePin()`, `exit()`, `returnCard()`
- **Entry Message**: "enter your card pin number"

#### SelectOperationState
- **Purpose**: Display and select transaction type
- **Responsibilities**:
  - Show available operations (CASH_WITHDRAWAL, BALANCE_CHECK)
  - Route to CashWithdrawalState or CheckBalanceState
  - Handle invalid selections
  - Return card on exit
- **Valid Operations**: `selectOperation()`, `exit()`, `returnCard()`
- **Entry Message**: "Please select the Operation"

#### CashWithdrawalState
- **Purpose**: Process cash withdrawal transaction
- **Responsibilities**:
  - Validate ATM balance (sufficient funds check)
  - Validate user balance (sufficient funds check)
  - Deduct amount from user's bank account
  - Deduct amount from ATM balance
  - Create and execute Chain of Responsibility for note distribution
  - Return card and transition to IdleState
- **Valid Operations**: `cashWithdrawal()`, `exit()`, `returnCard()`
- **Entry Message**: "Please enter the Withdrawal Amount"

#### CheckBalanceState
- **Purpose**: Display account balance
- **Responsibilities**:
  - Retrieve and display balance from card
  - Return card and transition to IdleState
- **Valid Operations**: `displayBalance()`, `exit()`, `returnCard()`
- **Entry Message**: None (constructor has no message)

### Chain of Responsibility Components

#### CashWithdrawProcessor (Abstract Handler)
- **Purpose**: Define interface for cash withdrawal chain
- **Responsibilities**:
  - Hold reference to next processor in chain
  - Define withdraw operation
  - Pass remaining amount to next processor if balance > 0
- **Pattern Role**: Handler in Chain of Responsibility
- **Key Field**: `nextCashWithdrawalProcessor`

#### TwoThousandWithdrawProcessor
- **Purpose**: Process ₹2000 notes (first in chain)
- **Responsibilities**:
  - Calculate required ₹2000 notes: `required = amount / 2000`
  - Calculate remaining balance: `balance = amount % 2000`
  - Deduct notes from ATM (up to available count)
  - If ATM has insufficient ₹2000 notes, adjust balance
  - Pass remaining balance to next processor (FiveHundredWithdrawProcessor)
- **Algorithm**: Greedy approach (use maximum denomination first)
- **Next Handler**: FiveHundredWithdrawProcessor

#### FiveHundredWithdrawProcessor
- **Purpose**: Process ₹500 notes (second in chain)
- **Responsibilities**:
  - Calculate required ₹500 notes: `required = amount / 500`
  - Calculate remaining balance: `balance = amount % 500`
  - Deduct notes from ATM (up to available count)
  - If ATM has insufficient ₹500 notes, adjust balance
  - Pass remaining balance to next processor (OneHundredWithdrawProcessor)
- **Algorithm**: Greedy approach
- **Next Handler**: OneHundredWithdrawProcessor

#### OneHundredWithdrawProcessor
- **Purpose**: Process ₹100 notes (last in chain)
- **Responsibilities**:
  - Calculate required ₹100 notes: `required = amount / 100`
  - Calculate remaining balance: `balance = amount % 100`
  - Deduct notes from ATM (up to available count)
  - If balance remains after processing, print error message
  - No next processor (end of chain)
- **Algorithm**: Greedy approach
- **Next Handler**: null (terminator)
- **Error Handling**: Prints "Something went wrong" if balance cannot be fulfilled

### User & Account Components

#### User
- **Purpose**: Represent ATM user
- **Responsibilities**:
  - Hold card reference
  - Hold bank account reference
- **Data Structure**: Simple container class

#### Card
- **Purpose**: Represent user's ATM card
- **Responsibilities**:
  - Store PIN number (static: 112211)
  - Validate entered PIN
  - Link to user's bank account
  - Provide balance information via bank account
  - Deduct balance from bank account
- **Key Methods**:
  - `isCorrectPINEntered(int pin)`: Validates PIN
  - `getBankBalance()`: Returns account balance
  - `deductBankBalance(int amount)`: Withdraws from account

#### UserBankAccount
- **Purpose**: Represent user's bank account
- **Responsibilities**:
  - Maintain account balance
  - Process withdrawal (deduct balance)
  - Provide balance information
- **Key Methods**:
  - `withdrawalBalance(int amount)`: Deducts amount
  - `getBalance()`: Returns current balance
  - `setBalance(int balance)`: Initializes balance

### Supporting Components

#### TransactionType (Enum)
- **Purpose**: Define available transaction types
- **Values**: 
  - `CASH_WITHDRAWAL`: Withdraw cash
  - `BALANCE_CHECK`: Check account balance
- **Method**: `showAllTransactionTypes()`: Displays all types

#### ATMRoom (Main Application)
- **Purpose**: Bootstrap and demonstrate ATM system
- **Responsibilities**:
  - Initialize ATM with balance and notes
  - Create user with card and bank account
  - Simulate transaction flow
  - Display ATM status
- **Methods**:
  - `initialize()`: Sets up ATM and user
  - `createUser()`: Factory method for user
  - `createCard()`: Factory method for card
  - `createBankAccount()`: Factory method for account

## Relationship Types

| From | To | Relationship | Type | Description |
|------|-----|--------------|------|-------------|
| ATMState | IdleState, HasCardState, SelectOperationState, CashWithdrawalState, CheckBalanceState | Inheritance | extends | State pattern hierarchy |
| CashWithdrawProcessor | TwoThousandWithdrawProcessor, FiveHundredWithdrawProcessor, OneHundredWithdrawProcessor | Inheritance | extends | Chain of Responsibility hierarchy |
| ATM | ATMState | Composition | contains | ATM owns its state |
| CashWithdrawProcessor | CashWithdrawProcessor | Aggregation | next | Chain linking |
| CashWithdrawalState | CashWithdrawProcessor | Dependency | uses | Creates and uses chain |
| User | Card | Composition | has | User owns card |
| User | UserBankAccount | Composition | has | User owns account |
| Card | UserBankAccount | Association | references | Card links to account |
| IdleState | HasCardState | Dependency | creates | State transition |
| HasCardState | SelectOperationState | Dependency | creates | State transition |
| SelectOperationState | CashWithdrawalState | Dependency | creates | State transition |
| SelectOperationState | CheckBalanceState | Dependency | creates | State transition |
| ATMRoom | ATM | Association | uses | Main uses ATM |
| ATMRoom | User | Association | uses | Main uses User |

## Key Design Insights

### Strengths
1. **Dual Pattern Implementation**: Effectively combines State and Chain of Responsibility patterns
2. **Clear Separation**: State management and cash distribution are independent concerns
3. **Extensibility**: Easy to add new states (e.g., DepositState, TransferState) or note denominations
4. **Security**: PIN validation before any transaction
5. **Singleton ATM**: Single point of control for all operations
6. **Greedy Algorithm**: Efficient note distribution (largest denominations first)
7. **Error Handling**: Validates both ATM and user balances before withdrawal

### Current Limitations
1. **Static PIN**: PIN hardcoded in Card class (112211)
2. **No transaction history**: No logging or audit trail
3. **No receipt generation**: Missing receipt functionality
4. **Limited error messages**: Generic "Something went wrong" messages
5. **No retry mechanism**: Invalid PIN leads to immediate exit (no 3-attempt rule)
6. **No optimal change algorithm**: Uses greedy approach (may fail in edge cases)
7. **Single card**: System designed for one card/user at a time
8. **No card blocking**: No mechanism to block card after multiple failed attempts

### Potential Enhancements

#### 1. Strategy Pattern for Change Algorithm
```java
interface WithdrawalStrategy {
    Map<Integer, Integer> calculateNotes(int amount, ATM atm);
}

class GreedyWithdrawalStrategy implements WithdrawalStrategy { }
class OptimalWithdrawalStrategy implements WithdrawalStrategy { }
```

#### 2. Observer Pattern for Notifications
```java
interface TransactionObserver {
    void onTransactionComplete(Transaction txn);
}

class SMSNotifier implements TransactionObserver { }
class EmailNotifier implements TransactionObserver { }
class AuditLogger implements TransactionObserver { }
```

#### 3. Factory Pattern for Card Types
```java
interface CardFactory {
    Card createCard();
}

class DebitCardFactory implements CardFactory { }
class CreditCardFactory implements CardFactory { }
class PrepaidCardFactory implements CardFactory { }
```

#### 4. Decorator Pattern for Transaction Fees
```java
abstract class TransactionDecorator {
    protected Transaction transaction;
    abstract int calculateFee();
}

class ServiceChargeDecorator extends TransactionDecorator { }
class TaxDecorator extends TransactionDecorator { }
```

#### 5. Command Pattern for Transaction History
```java
interface Command {
    void execute();
    void undo();
}

class WithdrawCommand implements Command { }
class DepositCommand implements Command { }
```

