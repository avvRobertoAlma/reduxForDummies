# reduxForDummies
Si tratta di una guida semplice per utilizzare Redux (pensata per essere compresa da utenti non professionisti)
# REDUX

## Base di Redux

### Installare Redux

Per installare **redux** digitare in una finestra di terminale: `yarn add redux` nella directory della web app.

### Creare uno store

Creare un file `store.js` e importare `{createStore}` da redux, come sotto indicato:

```javascript
import {createStore} from 'redux'

```
> con ES6 si può importare solo un elemento di un oggetto complesso utilizzando le parentesi graffe

Importare poi i **reducer** dalla cartella apposita.

Per creare uno store, si utilizza la sintassi:

`export default createStore(reducer)`

> in questo caso è chiaro che create store accetta come argomento un reducer, per cui dovremo creare un reducer (vedi sotto) ed importarlo nello store.

Lo store deve essere importato nell'entry point dell'applicazione. Es., potrebbe essere importato all'interno di **index.js**. Per caricare lo stato attuale, si inizializza una variabile con `store.getState()`.

Poi si passa la variabile `state` all'App, utilizzando l'espansione degli oggetti di ES6:

```javascript
<App {...state} />
```
oppure si passano i singoli elementi dello **state** come **props**

```javascript
<App todos={state.todos} />
```

### Creare un reducer
Il reducer è una funzione che combina lo stato con una `action` e restituisce il nuovo stato o lo stato precedente, a seconda della tipologia di azione.

Nel **reducer** solitamente si esporta una funzione che accetta due argomenti:

* il primo è lo stato (*solitamente viene passato uno stato di default consistente in un oggetto vuoto*)
* il secondo è l'azione.

Ecco un esempio di reducer:

```javascript
const initialState = {};
export default (state = initialState, action)
```
Detto ciò, si utilizza all'interno del corpo della funzione uno `switch`, dove ogni **case** è il relativo valore di `action.type`.

Esempio:

```javascript
switch (action.type) {
	case 'TODO_ADD': 
		return {...state, todos: state.todos.concat(action.payload)
	default:
		return state
```

#### Combinare più reducers

Abbiamo visto che nello `store` dobbiamo chiamare `export default createStore(reducer)`.

La funzione `createStore` accetta un solo `reducer`. Se dovessimo creare più di un `reducer` (ad esempio, un `messageReducer` e un `todoReducer`) dobbiamo utilizzare il metodo `combineReducer` di **redux**.

`import { combineReducers } from 'redux'`

A questo punto creo una nuova variabile `reducer` attraverso l'utilizzo di `combineReducers`. 

```javascript
const reducer = combineReducer({
	todo: todoReducer,
	message: messageReducer
})
```

Tutto ciò che è gestito da `todoReducer` sarà accessibile tramite `state.todo` es. `state.todo.todos`.

> Se si utilizza successivamente il combineReducers, ricordare di aggiornare l'applicazione. Es. una chiamata precedente a this.props.todos, dovrà essere modificata in this.props.todo.todos.

### Dispatching
Il **dispatch** consiste nell'invio di un'azione allo store.

Verrà quindi invocato il reducer e, quindi, verrà aggiornato lo stato.

Ad esempio:

```javascript
store.dispatch({type: 'TODO_ADD', 
payload: {id:4, name: 'redux dispatch, isComplete: false}})
```

#### Utilizzare gli *Action creators*

La classica sintassi per inviare un'azione è quella *inline*, ossia, quella che abbiamo riportato sopra.

Tuttavia, è possibile che le azioni siano utilizzate in più componenti e, pertanto, si propone un *pattern* basato sulle funzioni cd. *action creators*.

Si tratta, semplicemente di una funzione che restituisce un oggetto, contenente l'azione. 

Riprendendo l'esempio di prima, avremo:

```javascript
export const addTodo = (val) => ({ type: 'TODO_APP', payload: val})
```
A questo punto, questa *action creator* potrà essere importata in un altro componente e, per esempio, essere passata come argomento di `store.dispatch()`

Esempio:

```javascript
const newTodo = (todo) => {
	store.dispatch(addTodo(todo))
}

```


### Utilizzare react-redux


Installare con `yarn add react-redux`.

#### Il Provider

Effettuare il seguente import:

`import { Provider } from 'react-redux'`

il componente Provider si occupa già di gestire lo `store.subscribe()` ossia le eventuali modifiche da applicare alla UI in caso di aggiornamenti dello state.

> N.B. in React ogni aggiornamento dello state effettua una implicita chiamata a this.forceUpdate() e viene nuovamente effettuato il rendering. Con redux, occorre effettuare manualmente il forceUpdate(), generalmente nel ComponentDidMount lifecycle method.

**Provider** accetta come **prop** uno *store* e deve *wrappare* l'intera App, come da esempio seguente:

```javascript
<Provider store={store}>
	<App />
</Provider>
```

**Provider** renderà lo store disponibile per tutti i Componenti attraverso il **context** di React (anche se solitamente è una funzionalità non particolarmente incentivata)

#### Il metodo connect

Viene utilizzato per generare componenti container 

Una volta che abbiamo utilizzato il **Provider** possiamo utilizzare il metodo **connect**

`import { connect } from 'react-redux`

Questo metodo serve per collegare un componente di livello inferiore allo store.

Una maniera per collegare il component low-level allo store è la seguente:

```javascript
const ConnectedApp = connect()(App)
export default ConnectedApp
```


#### mapStateToProps

Tra i vari argomenti che **connect** accetta c'è `mapStateToProps` è una funzione che accetta come argomento l'intero **state** e restituisce come oggetto l'intero **state** o una parte di esso (a seconda delle nostre esigenze). 

`mapStateToProps` diventa quindi un argomento da passare al metodo `connect()` che renderà disponibile l'oggetto restituito come `props` del componente che abbiamo connesso allo store (che sarà quindi utilizzabile di default come qualsiasi `props` passata da un componente superiore).

Ad esempio se lo `state` fosse composto da `{ todos: [ ], users: [ ] }` e noi avessimo solo bisogno dell'array `todos`, la nostra **mapStateToProps** avrà la seguente struttura:

```javascript
const mapStateToProps = (state) => { todos: state.todos }

```

#### mapDispatchToProps

Il secondo argomento che **connect** accetta è `mapDispatchToProps`. Si tratta di una funzione che restituisce un oggetto che mapperà una determinata `prop` che consentirà di effettuare il `dispatch` di un'azione.

Esempio:

```javascript
const mapDispatchToProps = dispatch => ({
  addItem: val =>
    dispatch({
      type: "ADD_TODO",
      payload: {
        id: Date.now(),
        name: val,
        isComplete: false,
      },
    }),
})
```
Nell'esempio abbiamo dichiarato `mapDispatchToProps` come una funzione che accetta come primo argomento `dispatch`e restituisce un oggetto.

Questo oggetto ha un metodo `addItem` che accetta un argomento che verrà passato come valore della proprietà `name`. Il metodo `addItem` effettua finalmente la chiamata a `dispatch` e quindi invia l'azione allo `store`.

> esiste una sintassi più breve. Sufficiente inserire l'action creator come semplice oggetto es. { fetchTodos }

**Come utilizzare il metodo addItem?**

Molto semplicemente, nel nostro component potremo accedere al metodo con `this.props.addItem()`.


## Redux e i server

### JSON mock server (per testing)

Installare `json-server` con **yarn**.

Aggiungere un file `*.json` contenente i dati che si vogliono inserire nel server.

Aggiungere uno **script** a `package.json`:

```javascript
"dev-server": "json-server -p 3005 db.json"
```

Questo comando `dev-server` lancerà il nostro `json-server` sulla porta 3005 e chiamerà i dati contenuti in `db.json` (N.B. deve essere nella cartella principale dell'applicazione).

### Redux-thunk
Di default **redux** non consente di utilizzare **creatori di azioni** che restituiscano funzioni. Per questo occorre utilizzare `redux-thunk`. Installare con **yarn**.

#### Configurare redux-thunk

Editare lo **store**, importando anche `{ applyMiddleware }` da **redux**.
Importare quindi `thunk` da `redux-thunk`.

All'interno di **createStore**, quindi, chiamiamo come secondo argomento: `applyMiddleware(thunk`

### Creare una funzione per effettuare richieste GET

Creare una nuova directory chiamata, per convenzione, `lib`. All'interno creare un file `api.js` che conterrà le funzioni per interagire con il server.

```javascript
export const getTodos = () => {
  return fetch("http:localhost:3005/todos").then(res => res.json())
}
```

> questa funzione andrà utilizzata nel nostro reducer (importata nel file contenente i reducer).

#### Aggiornare il reducer
Nel reducer, è necessario creare un nuovo **action creator** che restituisca una funzione, con unico argomento `dispatch` che, a quel punto:

* prima deve effettuare una chiamata alla funzione GET che avevamo inserito nelle nostre API (ossia la funzione `getTodos()` che effettua una chiamata GET al nostro server e restituisce dati in formato JSON);
* poi, nel `.then` i dati in formato JSON saranno passati al metodo dispatch che a sua volta passerà i dati ottenuti dal server come `payload` dell'azione.

Esempio completo:

```javascript
export const loadTodos = todos => ({
  type: "LOAD_TODOS",
  payload: todos,
})

export const fetchTodos = () => {
  return dispatch => {
    getTodos().then(todos => dispatch(loadTodos(todos)))
  }
}
```

### Creare una richiesta POST

Nel file `api.js` creare una nuova funzione per inviare dati al server.

```javascript
export const addTodo = name => {
  return fetch("http://localhost:3005/todos", {
    method: "POST",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      id: Date.now(),
      name: name,
      isComplete: false,
    }),
  }).then(res => res.json())
}

```
> questa funzione andrà utilizzata nel nostro reducer (importata nel file contenente i reducer)

#### Aggiornare il reducer

Per gestire correttamente le nostre richieste POST, dovremmo:

* aggiungere un **action creator**;
* creare una nuova funzione che accetti un argomento e restituisca a sua volta una funzione che accetti come argomento `dispatch` ed invii l'azione creata prima allo `store`.

Questo è il codice dell'**action creator**:

```javascript
export const createTodo = todo => ({
  type: "ADD_TODO",
  payload: todo,
})
```

Si tratta di un'azione di tipo **ADD_TODO** che crea un nuovo elemento nella nostra lista.

Questa azione, tuttavia, deve essere eseguita in modo asincrono, in quanto deve inviare dati al server.

Creiamo, quindi, una nuova funzione che chiameremo `saveTodo` che accetta un argomento (ossia i dati da inviare al server) e restituisce una nuova funzione che, accetta come argomento `dispatch` e, successivamente:

1. Effettua la chiamata alle API per l'invio dei dati passati come argomento;
2. restituisce una `Promise`;
3. utilizziamo il `.then` e passiamo la `response` come argomento della funzione `createTodo()` che, a sua volta, viene passata a `dispatch` e che, finalmente, invia l'azione. 

Questo è il codice:

```javascript
export const saveTodo = name => {
  return dispatch => {
    addTodo(name).then(res => dispatch(createTodo(res)))
  }
}
```

A questo punto, possiamo importare questa ultima funzione `saveTodo` in tutti i componenti nei quali ci potrà essere utile.

