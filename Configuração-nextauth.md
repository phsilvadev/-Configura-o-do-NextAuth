# Configuração do NextAuth

!["nextauth"](/others/nextauth.png)

### 1. Instalação do Pacote

Execute o comando abaixo para instalar o pacote NextAuth:

```bash
npm install next-auth
```

### 2. Adicionar Rotas API

Criar a Rota de Autenticação

Crie o arquivo app/api/auth/[...nextauth]/route.ts com o seguinte conteúdo:

```bash
// @ts-nocheck
import axios from "@/shared/lib/axios";
import NextAuth from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        username: { label: "Username", type: "text", placeholder: "jsmith" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials, req) {
        const res = await axios.post("/auth/login", {
          user: credentials?.user,
          password: credentials?.password,
        });

        if (res.data) {
          return res.data;
        }
        return null;
      },
    }),
  ],
  session: {
    strategy: "jwt",
    maxAge: 60 * 60 * 2,
  },
  pages: {
    signIn: "/api/auth/signin",
    error: "/api/auth/signin",
  },
  callbacks: {
    async session({ session, token, user }) {
      session.user = token;
      return session;
    },
    async jwt({ token, account, user, trigger, session }) {
      if (trigger === "update") {
        token.access_token = session.user.access_token;
        token.refresh_token = session.user.refresh_token;
      }

      return { ...token, ...user };
    },
  },
});

export { handler as GET, handler as POST };
```

### 3. Criar a Página de Login

Crie o arquivo app/api/auth/signin/page.tsx com o seguinte conteúdo:

```bash
"use client";

import { useState } from "react";
import { signIn } from "next-auth/react";

const SignIn = () => {
  const [user, setUser] = useState<string>('');
  const [password, setPassword] = useState<string>('');

  const handleSignIn = () => {
    signIn("credentials", {
      username: user,
      password: password,
      redirect: true,
      callbackUrl: "/",
    });
  };

  return (
    <div>
      <input onChange={(e) => setUser(e.target.value)} type="text" placeholder="Usuário" />
      <input onChange={(e) => setPassword(e.target.value)} type="password" placeholder="Senha" />
      <button onClick={handleSignIn}>Entrar</button>
    </div>
  );
};

export default SignIn;

```

### 4. Middleware de Autenticação

Crie um arquivo chamado middleware.ts na raiz do projeto com o seguinte conteúdo:

```bash
import withAuth, { NextRequestWithAuth } from "next-auth/middleware";
import { User } from "./shared/@types/User";
import { headers } from "next/headers";

export default withAuth(
  function middleware(req) {
    return null;
  },
  {
    callbacks: {
      authorized: ({ token, req }) => {
        const headersList = headers();

        // Adicione outras rotas aqui que não precisam de token
        const routesWithoutToken = [];

        if (routesWithoutToken.includes(req.nextUrl.pathname)) {
          return true;
        }

        if (token) {
          const user = token as User;

          if (user.isAdmin) {
            return true;
          } else if (req.nextUrl.pathname.startsWith("/chamada/pagina-principal")) {
            return false;
          }
        }

        return false;
      },
    },
  }
);

```

### 5. Configuração dos Providers

Crie o arquivo app/providers.tsx com o seguinte conteúdo:

```bash
"use client";

import { SessionProvider } from "next-auth/react";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <SessionProvider>
      {children}
    </SessionProvider>
  );
}

```

### 6. Atualizar o Layout

No arquivo layout.ts, importe o SessionProvider:

```bash
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";
import { Providers } from "./providers";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "Exemplo",
  description: "",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="pt-br">
      <body suppressHydrationWarning={true} className={inter.className}>
        <Providers>
          <div className="h-screen w-screen">{children}</div>
        </Providers>
      </body>
    </html>
  );
}

```

### 7. Configuração do .env

Adicione as seguintes variáveis no arquivo .env:

```bash
NEXTAUTH_SECRET=your_secret_key
NEXTAUTH_URL=http://localhost:3000
```
