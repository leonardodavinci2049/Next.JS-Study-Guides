

# Guia Completo do Suspense no Next.js 15+

## 1. Introdução ao Suspense no Next.js

**O que é o Suspense?**  
Suspense é uma funcionalidade do React que permite "suspender" a renderização de um componente até que um recurso assíncrono (como dados de uma API) esteja pronto. Isso evita a renderização de estados incompletos da UI.

**Evolução no Next.js:**  
O Next.js 15 (e versões superiores) traz suporte completo ao Suspense de forma nativa com o App Router, integrando-o diretamente a funcionalidades como **Server Components**, **Streaming SSR**, **data fetching assíncrono**, e **transições de UI**.

**Diferenças do React puro para Next.js:**  
No React puro, Suspense é utilizado principalmente com `lazy()` e `React.Suspense` para importações dinâmicas. No Next.js App Router, ele é parte essencial do sistema de renderização assíncrona e do carregamento progressivo.

---

## 2. Conceitos Básicos

**Como funciona?**  
Quando um componente é "suspenso", ele avisa ao React que ainda não está pronto para renderizar. O React então exibe o conteúdo do `fallback` até que o componente possa ser exibido.

**Fallback:**  
Um UI alternativo, normalmente um loader ou skeleton.

### Exemplo simples com fetch assíncrono

```tsx
// app/components/Product.tsx
import { Suspense } from 'react'
import ProductData from './ProductData'

export default function Product() {
  return (
    <Suspense fallback={<div>Carregando produto...</div>}>
      <ProductData />
    </Suspense>
  )
}

// app/components/ProductData.tsx
export default async function ProductData() {
  const res = await fetch('https://api.exemplo.com/produto/1')
  const data = await res.json()

  return <div>{data.nome}</div>
}
````

---

## 3. Suspense com Server Components

**Uso:**
No Next.js 15, Server Components podem suspender automaticamente a renderização quando usam `fetch` com cache `force-cache`, `no-store`, ou `revalidate`.

**Vantagens:**

* Menor tempo de First Byte (TTFB)
* Carregamento progressivo via streaming

### Exemplo

```tsx
// app/page.tsx
import { Suspense } from 'react'
import ProdutosSection from './components/ProdutosSection'

export default function Page() {
  return (
    <div>
      <h1>Loja</h1>
      <Suspense fallback={<p>Carregando produtos...</p>}>
        <ProdutosSection />
      </Suspense>
    </div>
  )
}

// app/components/ProdutosSection.tsx
export default async function ProdutosSection() {
  const res = await fetch('https://api.exemplo.com/produtos')
  const produtos = await res.json()
  return (
    <ul>
      {produtos.map((p: any) => (
        <li key={p.id}>{p.nome}</li>
      ))}
    </ul>
  )
}
```

**Diferenças entre Server e Client Components:**

* Suspense em Server Components evita envio de JS desnecessário ao cliente.
* Em Client Components, Suspense precisa de fallback e depende de lazy-loading ou estados de carregamento.

---

## 4. Suspense com Streaming

**Streaming:**
Permite que partes da página sejam enviadas ao navegador conforme vão ficando prontas, sem precisar esperar o carregamento completo.

**Configuração:**
No App Router, isso é automático com `Suspense` + Server Components + `fetch` assíncrono.

### Exemplo

```tsx
export default function Home() {
  return (
    <main>
      <h1>Dashboard</h1>
      <Suspense fallback={<p>Carregando estatísticas...</p>}>
        <StatsSection />
      </Suspense>
    </main>
  )
}
```

**Melhores práticas:**

* Dividir seções que carregam dados pesados com `Suspense`.
* Usar fallbacks coerentes e visuais suaves (ex.: skeletons).

---

## 5. Uso Avançado do Suspense

**Múltiplos boundaries:**
Permite carregar blocos independentes da UI separadamente.

**Dependências aninhadas:**
Evite dependências em cascata que atrasam o carregamento geral.

**Integração com React Query/SWR:**
Essas libs agora suportam Suspense nativamente.

```tsx
// usando React Query
const { data } = useQuery(['produto', id], fetchProduto, {
  suspense: true
})
```

**Exemplo avançado:**
Página com transições assíncronas e Suspense.

```tsx
"use client"
import { useTransition } from 'react'

const [isPending, startTransition] = useTransition()

<button onClick={() => startTransition(() => router.push('/produto/2'))}>
  Ver Produto
</button>
```

---

## 6. Possíveis Armadilhas e Soluções

* **Waterfall**: múltiplos awaits sequenciais podem causar lentidão. Use paralelismo.
* **Fallback pobre**: usar só "Carregando..." pode causar sensação de travamento.
* **Erros silenciosos**: use `error.tsx` ou `ErrorBoundary` para tratar falhas.
* **Loop infinito**: evitar `useEffect` mal configurado em Client Component suspenso.

---

## 7. Melhores Práticas

* Use `Suspense` para carregamento progressivo, evitando estados `loading` manuais quando possível.
* Reutilize componentes com fallbacks claros e reutilizáveis.
* Otimize UX com placeholders visuais adequados.
* Evite Suspense onde o conteúdo é crítico para SEO.

---

## 8. Integrações e Casos Reais

**Integração com App Router:**
Suspense funciona nativamente com App Router e Server Components.

**Casos reais:**

* Dashboard com estatísticas e gráficos assíncronos
* E-commerce: carregamento de produtos, promoções e avaliações separadamente
* Blog: posts e comentários com carregamento independente

**Dynamic Imports + Suspense:**

```tsx
import dynamic from 'next/dynamic'

const Map = dynamic(() => import('./Map'), {
  ssr: false,
  loading: () => <p>Carregando mapa...</p>
})
```

---

## 9. Futuro do Suspense no Next.js

* Maior integração com frameworks como Relay e React Query.
* Ferramentas melhores de depuração para boundaries.
* Suporte expandido para cenários híbridos (RSC + Client).

**Referências:**

* [https://nextjs.org/docs/app/building-your-application/routing/loading-ui](https://nextjs.org/docs/app/building-your-application/routing/loading-ui)
* [https://react.dev/reference/react/Suspense](https://react.dev/reference/react/Suspense)
* [https://beta.reactjs.org/apis/react/Suspense](https://beta.reactjs.org/apis/react/Suspense)
