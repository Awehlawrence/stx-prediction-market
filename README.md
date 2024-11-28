# Stacks Prediction Market Contract

The **Stacks Prediction Market Contract** is a smart contract designed for deploying prediction markets on the Stacks blockchain. It enables users to create prediction events, place bets on event outcomes, resolve events, and distribute winnings based on the outcomes.

---

## Features

- **Event Creation**: Allows users to create prediction events with descriptions, options, and a resolution time.
- **Bet Placement**: Enables participants to place bets on specified event options.
- **Event Resolution**: Allows the contract owner to declare the winning outcome of an event.
- **Claim Winnings**: Distributes winnings to users who correctly bet on the winning outcome.
- **Event Cancellation**: Permits event creators to cancel events under certain conditions.
- **Refund System**: Provides refunds for canceled events.
- **Odds Calculation**: Dynamically calculates betting odds for each option.

---

## Key Constants

- `fee-percentage`: Defines the percentage fee deducted from the pool.
- `fee-denominator`: Used in conjunction with `fee-percentage` to compute precise fees.
- Various error codes for robust error handling (e.g., `err-unauthorized`, `err-invalid-bet`).

---

## Data Structures

### Data Maps

1. **Events**
    Stores event details, such as description, options, resolution time, and status.

    ```clarity
    (define-map events 
      { event-id: uint }
      {
         description: (string-ascii 256),
         options: (list 10 (string-ascii 64)),
         total-bets: uint,
         is-resolved: bool,
         winning-option: (optional uint),
         resolution-time: uint,
         creator: principal,
         is-canceled: bool
      }
    )
    ```

2. **Bets**
    Tracks bets placed by users for specific events and options.

    ```clarity
    (define-map bets
      { event-id: uint, better: principal }
      {
         amount: uint,
         option: uint
      }
    )
    ```

3. **Event Odds**
    Maps each event's option to its calculated odds.

    ```clarity
    (define-map event-odds
      { event-id: uint, option: uint }
      { odds: uint }
    )
    ```

---

## Core Functionalities

### Event Management

#### Create an Event

```clarity
(define-public (create-event (description (string-ascii 256)) (options (list 10 (string-ascii 64))) (resolution-time uint))
```

- **Inputs**: Description, options, and resolution time.
- **Returns**: The unique event ID.
- **Errors**: `err-invalid-bet` if invalid options or time.

#### Cancel an Event

```clarity
(define-public (cancel-event (event-id uint))
```

- **Requirements**:
  - Event creator must call this.
  - No bets must have been placed.
- **Effect**: Marks the event as canceled.

### Betting

#### Place a Bet

```clarity
(define-public (place-bet (event-id uint) (option uint) (amount uint))
```

- **Requirements**:
  - Event must be active.
  - Bet amount must be greater than zero.
  - Valid option ID must be provided.
- **Effect**: Deducts STX from the better and updates the event's total bet amount.

#### Refund Bet

```clarity
(define-public (refund-bet (event-id uint))
```

- **Effect**: Allows users to withdraw their bet amount if the event is canceled.

### Event Resolution and Payouts

#### Resolve Event

```clarity
(define-public (resolve-event (event-id uint) (winning-option uint))
```

- **Only callable by**: Contract owner.
- **Effect**: Declares the winning option for the event.

#### Claim Winnings

```clarity
(define-public (claim-winnings (event-id uint))
```

- **Requirements**:
  - Event must be resolved.
  - Caller must have bet on the winning option.
- **Effect**: Calculates and transfers winnings.

### Read-Only Functions

- **Get Event Details**: 

  ```clarity
  (define-read-only (get-event (event-id uint))
  ```

- **Get Bet Details**: 

  ```clarity
  (define-read-only (get-bet (event-id uint) (better principal)))
  ```

- **Calculate Odds**: 

  ```clarity
  (define-read-only (calculate-odds (event-id uint) (option uint)))
  ```

### Error Handling

Robust error codes ensure clear communication of issues:

- `err-unauthorized`: Caller lacks permission.
- `err-event-closed`: Bets placed after the resolution time.
- `err-invalid-option`: Invalid betting option.

---

## Testing

Implement unit tests for:

- Event creation, cancellation, and retrieval.
- Placing bets and claiming winnings.
- Edge cases such as invalid bets and unauthorized actions
