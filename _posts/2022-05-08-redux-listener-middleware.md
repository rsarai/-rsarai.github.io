---
layout: post
title: Experimenting with Redux Listener Middleware
categories: [frontend]
excerpt:
last_modified_at: 2022-05-09
---

For a while now, Redux is working on improving its image with the development of [Redux Toolkit](https://redux-toolkit.js.org/). The old conception that redux was too verbose, hard to onboard, hard to learn, and unnecessary for most applications is in fact becoming something of the past. The Redux Toolkit is full of utilities to ease development providing simpler ways to set up a store, create reducers, and deal with immutable updates, allowing you to do more with less code. Some highlight features are:
- RTK Query
- Typescript support
- createSlice
- createAsyncThunk
- createEntityAdapter
- createListenerMiddleware

I want to highlight the `createListenerMiddleware` today.

The listener middleware was created to be a lightweight layer to deal with side effects, other alternatives are redux-saga, thunks, and observables. The effects are callbacks with additional logic that can be triggered by other actions or state changes.

> Conceptually, you can think of this as being similar to React's useEffect hook, except that it runs logic in response to Redux store updates instead of component props/state updates.
> Redux Toolkit Documentation


## Problem
I recently worked on a web3 project where we didn't use Redux, only context to manage the global state. One of the challenges we had was: After a transaction is confirmed, we needed to trigger updates on all queries, to get the latest information and recalculate earnings. The calculations depended on the timestamp and block number, meaning that a lot could change from one block to another. The solution we used on the project was to set a global variable to store the block number of the fetched data for each route, when a transaction was made we would update this global value to make other places aware that they were out of sync. Trying to update a single tree of components was possible, but required some system thinking.

The problem appeared when trying to update information from an external API (not the blockchain) called The Graph which provides a graphQL API of indexed blockchain data. This API has a problem called Block Wobble: when data may appear to contradict itself. For example, when our queries to the blockchain return data for the block number 13, the API after a delay of a few seconds (due to distributed indexing) will send the data back to the user, however, because of the distributed notion of the blockchain the indexer may have to roll back from an uncle block and this can result in responses with weird block sequences in the client (this is a common concept in blockchain, [this link](https://thegraph.com/docs/en/developer/distributed-systems/) gives with a clear example).

Back to the refreshing problem, after a transaction one needs:
- Wait a few seconds to refetch information from The Graph API
- All queries are susceptible to block wobble
- There is a real possibility of the API lagging behind completely.

All these scenarios damage the side effects logic and can harm the user's experience. Given those issues, I decided to see how the listener middleware could handle those scenarios. In detail:

- Only persist a transaction to the state after 5 confirmations
- Wait a few seconds after persisting a transaction to trigger refetches
- Ignore intermediary states, if the transaction happened on block 13, don't save information from any other block (to deal with block wobble)



## Code
To create a sample project I used the template recommended on redux docs: `npx create-react-app my-app --template redux-typescript`


#### Scenario #1
Only mark a transaction as confirmed after 5 confirmations, ignoring intermediary states. In a real-world scenario confirmations would come from a Web Socket or would be continually fetched, for testing purposes I added a button to create fake confirmations.

The code required was simple: start listening for added transactions and create an effect to check if the transaction has enough confirmations. The listener API allows you to continually inspect the state for the number of confirmations, when the condition is met you can get details of the state to dispatch actions as you normally would.

```typescript
async function markTransactionAsComplete(
  action: AnyAction,
  listenerApi: AppListenerEffectAPI
) {
  // has enough confirmations to mark transaction as complete
  if (await listenerApi.condition(
    (action, currentState) =>
      currentState.transactions.current.confirmations.length >= 5)
  ) {
    const tx = listenerApi.getState().transactions.current.tx
    listenerApi.dispatch(
      transactionsActions.transactionComplete({tx})
    )
  }
}

startListening({
  actionCreator: transactionsActions.transactionAdd,
  effect: markTransactionAsComplete,
})
```

Notice you don't need to specify which actions to listen to (even though you could). The transaction is created, and the confirmations are added but only after the fifth one the transaction is marked as complete.

<img src="/images/2022-05-08-redux-listener-middleware/2022-05-08 11-58-redux-confirmed-after-5.gif" />

#### Scenario #2
Wait a few seconds after a transaction to trigger refetches. This one is the simplest, just add a delay and proceed.

```typescript
async function triggerRefetch(action: AnyAction, listenerApi: AppListenerEffectAPI) {
  // wait 3s to trigger a refetch
  await listenerApi.delay(3000)
  console.log("triggering refetch after 3s")
}


startListening({
  actionCreator: transactionsActions.transactionComplete,
  effect: triggerRefetch,
})
```

To account for The Graph I could trigger refetches indefinitely based on the block number of the answer, however, I would be wise to limit it to a few tries with exponential backoffs.

<img src="/images/2022-05-08-redux-listener-middleware/2022-05-08 11-58-redux-refetch-after-3s.gif" />

#### Scenario #3
Cancel all previous effects if something happens. The use case for this was to cancel the transaction if at least one confirmation doesn't come in a reasonable time, the cancelation is not only for the state but for all ongoing listeners.

```typescript
async function cancelIfStale(action: AnyAction, listenerApi: AppListenerEffectAPI) {
  // if first confirmation doesn't come in 10s, cancel transaction
  if (!await listenerApi.condition(
    (action, currentState) =>
      currentState.transactions.current.confirmations.length > 0, 10000)
  ) {
    const tx = listenerApi.getState().transactions.current.tx
    listenerApi.cancelActiveListeners()
    await listenerApi.dispatch(transactionsActions.transactionCancel({tx: tx}))
  }
}


startListening({
  actionCreator: transactionsActions.transactionAdd,
  effect: cancelIfStale,
})
```

<img src="/images/2022-05-08-redux-listener-middleware/2022-05-08 11-58-redux-cancel-after-10s.gif" />

--------
I found it easier, simpler and faster to trigger refetches with side effects instead of my previous way by manipulating state and re-renders.

## Links
- [Usage with typescript](https://redux-toolkit.js.org/usage/usage-with-typescript)
- [Matching Utilities](https://redux-toolkit.js.org/api/matching-utilities)
- [createListenerMiddleware](https://redux-toolkit.js.org/api/createListenerMiddleware)
- [Idiomatic Redux: Designing the Redux Toolkit Listener Middleware](https://blog.isquaredsoftware.com/2022/03/designing-rtk-listener-middleware/)
- [Examples: Action listener](https://github.dev/reduxjs/redux-toolkit/tree/master/examples/action-listener/counter)



