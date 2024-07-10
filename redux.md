# Configuração do Redxus na sua aplicação React

![redux](/others/redux.png)

O exemplo a seguir vai ser feito em nextjs mas pode ser aplicado em outros sem se nextjs

## Instalação

Serar nescessario a criação de dias lib

### Redux Toolkit

```bash
# NPM
npm install @reduxjs/toolkit

# Yarn
yarn add @reduxjs/toolkit
```

### Redux Core

```bash

# NPM
npm install redux

# Yarn
yarn add redux

```

## Exemplo de base

Criar um arquivo chamado src/app/store.js. Importar o configureStoreAPI do Redux Toolkit. Vamos começar criando uma loja Redux vazia, e exportando-a:

```bash
import { configureStore } from '@reduxjs/toolkit'

export default configureStore({
  reducer: {}
})
```

Isso cria uma loja Redux e também configura automaticamente a extensão Redux DevTools para que você possa inspecionar a loja enquanto se desenvolve.

## Forneça a Redux Store globalmente

Uma vez que a loja é criada, podemos torná-la disponível para nossos componentes React, colocando um React-Redux <Provider>em torno da nossa aplicação em src/index.js. Importe a loja Redux que acabamos de criar, colocar um <Provider>em torno de seu <App>E passar a loja como um adereço:

```bash
// app/providers.tsx
"use client";

import store from "@/shared/redux/storage";
import { NextUIProvider } from "@nextui-org/react";
import { SessionProvider } from "next-auth/react";
import { Provider } from "react-redux";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <Provider store={store}>
      <SessionProvider>
        <NextUIProvider>{children}</NextUIProvider>
      </SessionProvider>
    </Provider>
  );
}

```

## Criar um Redux State Slice

Adicionar um novo arquivo chamado src/features/sessionAuth.js Nesse arquivo, importe o createSliceAPI do Redux Toolkit.

no exemplo em baixo está mostrando ima features da sessão do usuario, vamos armazer o mesmo assim que ser logando no sistema

```bash

import { createSlice } from "@reduxjs/toolkit";

interface LoginPayload {
  id: number;
  Name: string;
  isActivate: boolean;
  isAdmin: boolean;
  access_token: string;
  refresh_token: string;
}

type PlayLoadAction = {
  payload: LoginPayload;
};

interface AuthState {
  id: number;
  Name: string;
  isActivate: boolean;
  isAdmin: boolean;
  access_token: string;
  refresh_token: string;
}

interface InitialState {
  value: AuthState;
}

const initialState = {
  value: {
    id: 0,
    Name: "",
    isActivate: false,
    isAdmin: false,
    access_token: "",
    refresh_token: "",
  } as AuthState,
} as InitialState;

const sessionAuthSlice = createSlice({
  name: "sessionAuth",
  initialState,
  reducers: {
    logOut: () => {
      return initialState;
    },
    logIn: (state, action: PlayLoadAction) => {
      return {
        value: {
          id: action.payload.id,
          Name: action.payload.Name,
          isAdmin: action.payload.isAdmin,
          isActivate: action.payload.isAdmin,
          access_token: action.payload.access_token,
          refresh_token: action.payload.refresh_token,
        },
      };
    },
  },
});

export const { logIn, logOut } = sessionAuthSlice.actions;
export default sessionAuthSlice.reducer;


```

## Storage

dentro do mesmo vamos importar a features sessionAuthSlice

```bash

import { configureStore } from "@reduxjs/toolkit";
import { TypedUseSelectorHook, useSelector } from "react-redux";
import sessionAuthSlice from "./features/sessionAuthSlice.ts";

const store = configureStore({
  reducer: {
    sessionAuthSlice
  },
});

export default store;

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;


```

## Inserir os Dados

```bash
...

export default function Admin() {


  const dispatch = useDispatch<AppDispatch>();
  const ticketWindow = useAppSelector((state) => state.ticketWindow.name);

  const { data: session, status } = useSession({
    required: true,
    onUnauthenticated() {
      // The user is not authenticated, handle it here.
    },
  });

  const typeUser = session?.user as User;

  useEffect(() => {
    if (typeUser != undefined) {
      dispatch(
        logIn({
          id: typeUser.id,
          access_token: typeUser.access_token,
          fullName: typeUser.fullName,
          isActivate: typeUser.isActivate,
          isAdmin: typeUser.isAdmin,
          user: typeUser.user,
          permissions: typeUser.permissions,
          refresh_token: typeUser.refresh_token,
          licence: typeUser.licence,
        })
      );
    }
  }, [typeUser, ticketWindow]);

  ...
}


```

## Chamado

Dados dos usuario inserido no redux, agora vamos pegar o mesmo

```bash
...
 const user = useAppSelector((state) => state.sessionUser.value);


 useEffect(()=>{
    console.log(user.id)
 },[user])
 ...
```
