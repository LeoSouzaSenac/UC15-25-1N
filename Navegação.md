# Navegação no React Native

> **Tecnologias:** Expo SDK 58 · React Native 0.88 · React 19.3 · TypeScript · React Navigation 7  
> **Objetivo:** criar aplicativos com várias telas, transmitir dados entre elas e combinar **Bottom Tabs + Native Stack**.  
> **Pré-requisitos:** componentes, props, `useState`, objetos, TypeScript e organização de arquivos com `import`/`export`.

## O que vamos aprender?

Ao final desta aula, você deverá conseguir:

1. Explicar a diferença entre navegação por **pilha (Stack)** e por **abas (Bottom Tabs)**.
2. Instalar e configurar o React Navigation em um projeto Expo SDK 58.
3. Criar telas em **arquivos separados**, sem usar `any`.
4. Abrir telas, voltar e enviar parâmetros tipados.
5. Criar abas inferiores com ícones.
6. **Combinar Stack e Bottom Tabs em um mesmo aplicativo**, com uma tela de detalhes aberta dentro de uma aba.

---

## 1. Por que um aplicativo precisa de navegação?

Pense em um aplicativo de compras: na tela principal, você vê vários produtos. Ao tocar em um produto, abre uma nova tela com o preço e a descrição. Ao tocar em **Voltar**, retorna à lista. Já na parte de baixo do app, talvez existam abas como **Início**, **Favoritos** e **Configurações**.

São duas situações diferentes:

| Navegação | Como funciona | Exemplo |
|---|---|---|
| **Native Stack** | Coloca uma tela sobre outra, como cartas empilhadas | Lista de produtos → Detalhes do produto → Voltar |
| **Bottom Tabs** | Permite trocar entre áreas do app tocando nas abas inferiores | Início ↔ Favoritos ↔ Configurações |

**Uma analogia:** o Stack funciona como uma pilha de folhas na mesa. As Tabs funcionam como divisórias de um fichário.

> **Importante:** um aplicativo React Native não depende do histórico de páginas de um navegador web. O React Navigation gerencia o estado e as ações de navegação. React Native também pode trabalhar com *deep links*, mas isso é assunto para outra aula.

## 2. Preparando o ambiente — Expo SDK 58

A SDK 58 utiliza **React Native 0.88 e React 19.3**. Para ferramentas Node, utilize uma versão compatível: **Node 22.13+**, **24.3+** ou **26+**.

Esta aula pressupõe um projeto que usa **`App.tsx` como ponto de entrada**, em vez de um projeto já configurado com Expo Router. Se você criou o projeto com um template que já usa a pasta `app/` e Expo Router, não misture os dois modelos de navegação no mesmo ponto de entrada: utilize um projeto com entrada tradicional para acompanhar os exemplos.

### 2.1. Instalando as bibliotecas

No terminal, **dentro da pasta do projeto**:

```bash
npm install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
```

As dependências nativas devem ser instaladas com o Expo, para resolver versões compatíveis com a SDK:

```bash
npx expo install react-native-screens react-native-safe-area-context
```

Para os ícones das abas:

```bash
npx expo install @expo/vector-icons
```

Confira possíveis incompatibilidades:

```bash
npx expo install --check
```

Se for preciso alinhar as dependências às versões compatíveis com o Expo:

```bash
npx expo install --fix
```

> **Por que `npx expo install`?** Bibliotecas com código nativo precisam de versões compatíveis com a versão do React Native utilizada pelo Expo. O comando escolhe as versões indicadas para a SDK do projeto.

> **Atenção ao Expo Go:** o anúncio da SDK 58, publicado em 15/10/2026, informou que a versão correspondente do Expo Go seria disponibilizada posteriormente nas lojas. Se o Expo Go instalado no aparelho ainda não suportar a SDK 58, utilize uma **development build** ou verifique a disponibilidade atual antes da aula.

---

# Parte A — Primeiro, vamos entender o Stack

## 3. Organizando os arquivos

Crie a seguinte estrutura:

```text
MeuApp/
├── App.tsx
├── screens/
│   ├── HomeScreen.tsx
│   └── ProfileScreen.tsx
└── types/
    └── navigation.ts
```

**Por que não colocar tudo no `App.tsx`?** Cada tela tem uma responsabilidade. Arquivos separados tornam a leitura, a manutenção e a reutilização muito mais simples.

## 4. Tipando as rotas

**Arquivo: `types/navigation.ts`**

```typescript
export type RootStackParamList = {
  Home: undefined;
  Profile: {
    userId: number;
    nome: string;
  };
};
```

**Como ler isso?**

- `Home: undefined` significa que a tela inicial **não precisa receber parâmetros**.
- `Profile: { userId: number; nome: string }` significa que, para abrir o perfil, devemos enviar **um número e um texto**.

Exemplo correto:

```typescript
{ userId: 10, nome: 'Maria' }
```

Exemplo incorreto:

```typescript
{ userId: 'dez', nome: 'Maria' } // userId precisa ser number
```

> Utilizamos `type`, não `interface`, para a lista de parâmetros. O React Navigation recomenda um *type alias* para essa definição.

## 5. Tela inicial

**Arquivo: `screens/HomeScreen.tsx`**

```tsx
import { View, Text, Button, StyleSheet } from 'react-native';
import type { NativeStackScreenProps } from '@react-navigation/native-stack';
import type { RootStackParamList } from '../types/navigation';

type Props = NativeStackScreenProps<RootStackParamList, 'Home'>;

export default function HomeScreen({ navigation }: Props) {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Tela Inicial</Text>
      <Text>Escolha um perfil:</Text>

      <Button
        title="Abrir perfil de Maria"
        onPress={() =>
          navigation.navigate('Profile', { userId: 10, nome: 'Maria' })
        }
      />

      <Button
        title="Abrir perfil de João"
        onPress={() =>
          navigation.navigate('Profile', { userId: 25, nome: 'João' })
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    gap: 20,
    padding: 20,
    backgroundColor: '#FFFFFF',
  },
  titulo: { fontSize: 26, fontWeight: 'bold' },
});
```

Observe:

```tsx
navigation.navigate('Profile', { userId: 10, nome: 'Maria' })
```

- `'Profile'` é o **nome da rota** registrada no navegador.
- O objeto `{ userId: 10, nome: 'Maria' }` contém os **dados enviados**.
- O TypeScript verifica se os parâmetros seguem os tipos declarados.

## 6. Tela de destino

**Arquivo: `screens/ProfileScreen.tsx`**

```tsx
import { View, Text, Button, StyleSheet } from 'react-native';
import type { NativeStackScreenProps } from '@react-navigation/native-stack';
import type { RootStackParamList } from '../types/navigation';

type Props = NativeStackScreenProps<RootStackParamList, 'Profile'>;

export default function ProfileScreen({ navigation, route }: Props) {
  const { userId, nome } = route.params;

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Perfil de {nome}</Text>
      <Text>ID do usuário: {userId}</Text>

      <Button title="Voltar" onPress={() => navigation.goBack()} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    gap: 20,
    padding: 20,
    backgroundColor: '#FFFFFF',
  },
  titulo: { fontSize: 26, fontWeight: 'bold' },
});
```

### Qual é a diferença entre `navigation` e `route`?

| Objeto | Para que serve? | Exemplo |
|---|---|---|
| `navigation` | Controla a navegação | `navigation.goBack()` |
| `route` | Identifica a tela e seus parâmetros | `route.params.nome` |

**`route.params`** recebe o objeto enviado pela tela anterior. **`navigation.goBack()`** tenta retornar à tela anterior na pilha.

## 7. Registrando as telas

**Arquivo: `App.tsx`**

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { SafeAreaProvider } from 'react-native-safe-area-context';

import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';
import type { RootStackParamList } from './types/navigation';

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <SafeAreaProvider>
      <NavigationContainer>
        <Stack.Navigator initialRouteName="Home">
          <Stack.Screen
            name="Home"
            component={HomeScreen}
            options={{ title: 'Início' }}
          />
          <Stack.Screen
            name="Profile"
            component={ProfileScreen}
            options={{ title: 'Perfil' }}
          />
        </Stack.Navigator>
      </NavigationContainer>
    </SafeAreaProvider>
  );
}
```

**O que faz cada parte?**

- `NavigationContainer`: gerencia o estado geral da navegação.
- `Stack.Navigator`: cria a navegação em pilha.
- `Stack.Screen`: registra uma tela pelo seu nome e componente.
- `initialRouteName="Home"`: escolhe qual rota é aberta primeiro.
- `SafeAreaProvider`: fornece informações sobre as áreas seguras do dispositivo.

Inicie o aplicativo:

```bash
npx expo start
```

**Teste:** toque em **Abrir perfil de Maria**, confira o nome e o ID e volte. Repita com João. Você criou duas experiências diferentes usando **uma única tela de perfil**.

### Exercício 1

Troque os dados enviados para um novo usuário, chamado Ana, com ID 42. Depois experimente enviar `userId: '42'` (string) e observe o erro indicado pelo TypeScript.

---

# Parte B — Navegação com Bottom Tabs

## 8. O que são abas inferiores?

As abas aparecem na parte de baixo do aplicativo e permitem alternar entre seções principais, como **Início**, **Favoritos** e **Configurações**.

Ao contrário do Stack, aqui normalmente não pensamos em "entrar" e "voltar", mas em **mudar de seção**.

Para praticar separadamente, vamos criar um segundo exemplo. Ele utiliza os mesmos pacotes já instalados.

### Estrutura do exemplo de Tabs

```text
ExemploTabs/
├── App.tsx
├── screens/
│   ├── InicioScreen.tsx
│   └── FavoritosScreen.tsx
└── types/
    └── navigation.ts
```

> **Atenção:** os exemplos das partes A, B e C são projetos independentes. Não copie três versões de `App.tsx` para o mesmo projeto. Na **Parte C**, criaremos o aplicativo final usando os dois navegadores juntos.

### 8.1. Tipos das abas

**Arquivo: `types/navigation.ts`**

```typescript
export type TabParamList = {
  Inicio: undefined;
  Favoritos: undefined;
};
```

### 8.2. Tela Início

**Arquivo: `screens/InicioScreen.tsx`**

```tsx
import { View, Text, StyleSheet } from 'react-native';

export default function InicioScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Início</Text>
      <Text>Bem-vindo ao app!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', gap: 12 },
  titulo: { fontSize: 26, fontWeight: 'bold' },
});
```

### 8.3. Tela Favoritos

**Arquivo: `screens/FavoritosScreen.tsx`**

```tsx
import { View, Text, StyleSheet } from 'react-native';

export default function FavoritosScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Favoritos</Text>
      <Text>Seus favoritos aparecerão aqui.</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', gap: 12 },
  titulo: { fontSize: 26, fontWeight: 'bold' },
});
```

### 8.4. App.tsx com Bottom Tabs

**Arquivo: `App.tsx`**

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';

import InicioScreen from './screens/InicioScreen';
import FavoritosScreen from './screens/FavoritosScreen';
import type { TabParamList } from './types/navigation';

const Tab = createBottomTabNavigator<TabParamList>();

export default function App() {
  return (
    <SafeAreaProvider>
      <NavigationContainer>
        <Tab.Navigator>
          <Tab.Screen
            name="Inicio"
            component={InicioScreen}
            options={{
              title: 'Início',
              tabBarIcon: ({ color, size }) => (
                <Ionicons name="home-outline" color={color} size={size} />
              ),
            }}
          />
          <Tab.Screen
            name="Favoritos"
            component={FavoritosScreen}
            options={{
              title: 'Favoritos',
              tabBarIcon: ({ color, size }) => (
                <Ionicons name="heart-outline" color={color} size={size} />
              ),
            }}
          />
        </Tab.Navigator>
      </NavigationContainer>
    </SafeAreaProvider>
  );
}
```

**Entendendo:** `Tab.Navigator` cria as abas; cada `Tab.Screen` representa uma seção. Em `tabBarIcon`, o React Navigation fornece a cor e o tamanho adequados para o ícone.

### Exercício 2

Crie uma terceira aba chamada **Configurações**. Ela deve ter um ícone de engrenagem (`settings-outline`) e mostrar o texto "Configurações do aplicativo". Faça isso com **um arquivo de tela separado**.

---

# Parte C — EXEMPLO COMPLETO: Bottom Tabs + Stack juntos

## 9. O aplicativo que vamos construir

Agora vamos fazer um **único aplicativo** em que os dois tipos de navegação trabalham juntos.

Imagine uma pequena loja:

- A barra inferior oferece **Início**, **Favoritos** e **Configurações**.
- Na aba **Início**, aparecem produtos.
- Ao tocar em um produto, abre a tela de **Detalhes do Produto** usando um **Native Stack**.
- Ao voltar, o usuário retorna para a lista de produtos.
- **As abas continuam visíveis mesmo na tela de detalhes**, porque o Stack está **dentro da aba Início**.

### 9.1. Como os navegadores estarão organizados

```text
NavigationContainer
└── Bottom Tab Navigator
    ├── Início [aba]
    │   └── Native Stack Navigator
    │       ├── Lista de Produtos  ← primeira tela da pilha
    │       └── Detalhes do Produto ← abre por cima da lista
    ├── Favoritos [aba]
    └── Configurações [aba]
```

**Essa é a ideia mais importante da aula:** as Tabs alternam entre áreas; dentro da aba Início, o Stack permite avançar e voltar entre telas.

> Outra arquitetura possível seria colocar o **Bottom Tabs dentro de um Stack principal**. Nesse caso, uma tela de detalhes aberta no Stack principal pode cobrir as abas. Não existe uma única organização obrigatória: escolhemos conforme a experiência desejada.

## 10. Arquivos do aplicativo final

Para evitar confusão com os exemplos anteriores, crie um novo projeto **com entrada `App.tsx`** ou reorganize um projeto compatível. Os arquivos finais serão:

```text
MinhaLoja/
├── App.tsx
├── data/
│   └── produtos.ts
├── navigation/
│   ├── InicioStack.tsx
│   └── AppTabs.tsx
├── screens/
│   ├── ProdutosScreen.tsx
│   ├── DetalhesScreen.tsx
│   ├── FavoritosScreen.tsx
│   └── ConfiguracoesScreen.tsx
└── types/
    └── navigation.ts
```

### 10.1. Primeiro: tipos das rotas

**Arquivo: `types/navigation.ts`**

```typescript
import type { NavigatorScreenParams } from '@react-navigation/native';

export type InicioStackParamList = {
  Produtos: undefined;
  Detalhes: { produtoId: number };
};

export type AppTabParamList = {
  Inicio: NavigatorScreenParams<InicioStackParamList>;
  Favoritos: undefined;
  Configuracoes: undefined;
};
```

**Por que `NavigatorScreenParams`?** Porque a aba `Inicio` não aponta diretamente para uma tela comum: ela contém **outro navegador**, com suas próprias rotas. O tipo `NavigatorScreenParams` informa ao TypeScript que existe uma navegação aninhada.

### 10.2. Criando dados de exemplo

Ainda não precisamos consumir uma API. Vamos utilizar um array simples.

**Arquivo: `data/produtos.ts`**

```typescript
export type Produto = {
  id: number;
  nome: string;
  preco: number;
  descricao: string;
};

export const produtos: Produto[] = [
  {
    id: 1,
    nome: 'Teclado Mecânico',
    preco: 249.9,
    descricao: 'Teclado para trabalhar, estudar e jogar.',
  },
  {
    id: 2,
    nome: 'Mouse Sem Fio',
    preco: 89.9,
    descricao: 'Mouse confortável para uso diário.',
  },
  {
    id: 3,
    nome: 'Headset',
    preco: 159.9,
    descricao: 'Fone com microfone para chamadas e jogos.',
  },
];
```

**Vantagem:** se quisermos adicionar um produto, basta adicionar outro objeto ao array. Não precisamos criar outra tela!

### 10.3. Criando a lista de produtos

**Arquivo: `screens/ProdutosScreen.tsx`**

```tsx
import { View, Text, FlatList, Pressable, StyleSheet } from 'react-native';
import type { NativeStackScreenProps } from '@react-navigation/native-stack';

import { produtos } from '../data/produtos';
import type { InicioStackParamList } from '../types/navigation';

type Props = NativeStackScreenProps<InicioStackParamList, 'Produtos'>;

export default function ProdutosScreen({ navigation }: Props) {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Produtos da loja</Text>
      <Text>Toque em um produto para ver os detalhes.</Text>

      <FlatList
        data={produtos}
        keyExtractor={(item) => item.id.toString()}
        contentContainerStyle={styles.lista}
        renderItem={({ item }) => (
          <Pressable
            style={styles.card}
            onPress={() =>
              navigation.navigate('Detalhes', { produtoId: item.id })
            }
          >
            <Text style={styles.nome}>{item.nome}</Text>
            <Text>R$ {item.preco.toFixed(2).replace('.', ',')}</Text>
            <Text style={styles.link}>Ver detalhes →</Text>
          </Pressable>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#F4F6F8' },
  titulo: { fontSize: 26, fontWeight: 'bold', marginBottom: 8 },
  lista: { paddingVertical: 16, gap: 12 },
  card: {
    backgroundColor: '#FFFFFF',
    padding: 18,
    borderRadius: 12,
    borderWidth: 1,
    borderColor: '#E1E5EA',
    gap: 8,
  },
  nome: { fontSize: 18, fontWeight: 'bold' },
  link: { color: '#1454BD', fontWeight: 'bold' },
});
```

Aqui, a lista utiliza `FlatList`, e cada produto recebe um `Pressable` para abrir os detalhes. O comando principal é:

```typescript
navigation.navigate('Detalhes', { produtoId: item.id });
```

**Perceba:** não enviamos o produto inteiro. Enviamos apenas seu **ID**. A tela de detalhes usa esse ID para buscar o produto no nosso array. Essa organização também será útil quando trabalharmos com APIs e bancos de dados.

### 10.4. Criando a tela de detalhes

**Arquivo: `screens/DetalhesScreen.tsx`**

```tsx
import { View, Text, Button, StyleSheet } from 'react-native';
import type { NativeStackScreenProps } from '@react-navigation/native-stack';

import { produtos } from '../data/produtos';
import type { InicioStackParamList } from '../types/navigation';

type Props = NativeStackScreenProps<InicioStackParamList, 'Detalhes'>;

export default function DetalhesScreen({ navigation, route }: Props) {
  const { produtoId } = route.params;
  const produto = produtos.find((item) => item.id === produtoId);

  if (!produto) {
    return (
      <View style={styles.container}>
        <Text>Produto não encontrado.</Text>
        <Button title="Voltar" onPress={() => navigation.goBack()} />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>{produto.nome}</Text>
      <Text style={styles.preco}>
        R$ {produto.preco.toFixed(2).replace('.', ',')}
      </Text>
      <Text>{produto.descricao}</Text>
      <Button title="Voltar para a lista" onPress={() => navigation.goBack()} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
    gap: 20,
    backgroundColor: '#FFFFFF',
  },
  titulo: { fontSize: 26, fontWeight: 'bold', textAlign: 'center' },
  preco: { fontSize: 22, color: '#1454BD', fontWeight: 'bold' },
});
```

O que acontece quando abrimos os detalhes?

1. O Stack mostra a tela `Detalhes`.
2. A tela recebe `produtoId` por meio de `route.params`.
3. O método `.find()` procura o produto no array.
4. As informações aparecem na tela.
5. `goBack()` retorna à lista.

### 10.5. Criando o Stack da aba Início

**Arquivo: `navigation/InicioStack.tsx`**

```tsx
import { createNativeStackNavigator } from '@react-navigation/native-stack';

import ProdutosScreen from '../screens/ProdutosScreen';
import DetalhesScreen from '../screens/DetalhesScreen';
import type { InicioStackParamList } from '../types/navigation';

const Stack = createNativeStackNavigator<InicioStackParamList>();

export default function InicioStack() {
  return (
    <Stack.Navigator initialRouteName="Produtos">
      <Stack.Screen
        name="Produtos"
        component={ProdutosScreen}
        options={{ title: 'Minha Loja' }}
      />
      <Stack.Screen
        name="Detalhes"
        component={DetalhesScreen}
        options={{ title: 'Detalhes do Produto' }}
      />
    </Stack.Navigator>
  );
}
```

**O detalhe mais importante:** este arquivo exporta **um navegador inteiro**, não apenas uma tela. Mais adiante, esse navegador será registrado como o conteúdo da aba `Inicio`.

### 10.6. Criando a tela Favoritos

**Arquivo: `screens/FavoritosScreen.tsx`**

```tsx
import { View, Text, StyleSheet } from 'react-native';

export default function FavoritosScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Favoritos</Text>
      <Text>Aqui ficarão seus produtos favoritos.</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', gap: 12 },
  titulo: { fontSize: 26, fontWeight: 'bold' },
});
```

### 10.7. Criando a tela Configurações

Vamos aproveitar para praticar `useState` e `Switch`.

**Arquivo: `screens/ConfiguracoesScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, Switch, StyleSheet } from 'react-native';

export default function ConfiguracoesScreen() {
  const [notificacoes, setNotificacoes] = useState(false);

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Configurações</Text>
      <Text>
        Notificações: {notificacoes ? 'Ativadas' : 'Desativadas'}
      </Text>
      <Switch value={notificacoes} onValueChange={setNotificacoes} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', gap: 16 },
  titulo: { fontSize: 26, fontWeight: 'bold' },
});
```

> Este `Switch` altera apenas o estado visual da tela. Ele **não ativa notificações reais do sistema**. Para isso, precisaríamos aprender permissões e uma biblioteca de notificações.

### 10.8. Criando as abas e colocando o Stack dentro delas

**Arquivo: `navigation/AppTabs.tsx`**

```tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Ionicons } from '@expo/vector-icons';

import InicioStack from './InicioStack';
import FavoritosScreen from '../screens/FavoritosScreen';
import ConfiguracoesScreen from '../screens/ConfiguracoesScreen';
import type { AppTabParamList } from '../types/navigation';

const Tab = createBottomTabNavigator<AppTabParamList>();

export default function AppTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen
        name="Inicio"
        component={InicioStack}
        options={{
          title: 'Início',
          headerShown: false,
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home-outline" color={color} size={size} />
          ),
        }}
      />

      <Tab.Screen
        name="Favoritos"
        component={FavoritosScreen}
        options={{
          title: 'Favoritos',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="heart-outline" color={color} size={size} />
          ),
        }}
      />

      <Tab.Screen
        name="Configuracoes"
        component={ConfiguracoesScreen}
        options={{
          title: 'Configurações',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="settings-outline" color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}
```

**Pare e observe estas duas linhas:**

```tsx
name="Inicio"
component={InicioStack}
```

Na aba Início, **não** registramos diretamente `ProdutosScreen`. Registramos `InicioStack`, que já sabe navegar entre `ProdutosScreen` e `DetalhesScreen`.

E por que usamos `headerShown: false` na aba Início? Porque o **Stack interno** já desenha seu próprio cabeçalho. Assim evitamos dois cabeçalhos sobrepostos.

### 10.9. App.tsx final — unindo tudo

**Arquivo: `App.tsx`**

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { SafeAreaProvider } from 'react-native-safe-area-context';

import AppTabs from './navigation/AppTabs';

export default function App() {
  return (
    <SafeAreaProvider>
      <NavigationContainer>
        <AppTabs />
      </NavigationContainer>
    </SafeAreaProvider>
  );
}
```

**Só isso!** A configuração completa foi organizada em arquivos pequenos:

- `App.tsx` cria o container da navegação.
- `AppTabs.tsx` cria a barra inferior.
- `InicioStack.tsx` cria a pilha **dentro** da aba Início.
- `ProdutosScreen.tsx` e `DetalhesScreen.tsx` cuidam das telas da pilha.

Esse é o **exemplo completo com Bottom Tabs e Stack funcionando juntos**, sem `any` e sem colocar tudo no mesmo arquivo.

## 11. Testando o aplicativo completo

Execute:

```bash
npx expo start
```

Teste nesta ordem:

1. Abra o app. Você deverá visualizar a aba **Início** com a lista de produtos.
2. Toque em **Teclado Mecânico**. A tela **Detalhes do Produto** deve abrir.
3. Confira que **a barra inferior continua aparecendo**.
4. Toque em **Voltar** no cabeçalho ou no botão da própria tela.
5. Abra outro produto. O nome, o preço e a descrição devem mudar.
6. Toque na aba **Favoritos**.
7. Toque na aba **Configurações** e altere o Switch.
8. Retorne à aba **Início**.

Se tudo isso funcionar, você configurou corretamente a navegação aninhada.

### Atenção: e se eu quisesse esconder as abas na tela de detalhes?

Nesse caso, poderíamos usar **a arquitetura inversa**:

```text
NavigationContainer
└── Native Stack Navigator
    ├── Tabs
    │   ├── Início
    │   ├── Favoritos
    │   └── Configurações
    └── Detalhes (fora das abas)
```

Aqui, a tela de detalhes pertence ao Stack principal e aparece **por cima do navegador de abas**. O resultado é uma tela de detalhes sem barra inferior. As duas arquiteturas são válidas; a escolha depende do comportamento desejado.

---

## 12. Principais ações de navegação

| Ação | Explicação |
|---|---|
| `navigation.navigate('Tela')` | Navega para uma rota registrada, seguindo as regras do navegador atual |
| `navigation.goBack()` | Volta quando existe uma tela anterior no histórico |
| `navigation.push('Tela', params)` | No Stack, empilha uma **nova instância** da rota |
| `navigation.replace('Tela', params)` | No Stack, substitui a rota atual |
| `navigation.popTo('Tela')` | No Stack, retorna a uma rota existente na pilha ou aplica o comportamento definido pela versão da biblioteca |
| `navigation.setParams({...})` | Atualiza parâmetros da rota atual |

Não confunda `navigate` com `push`:

- `navigate` obedece à lógica de navegação para a rota de destino. **Não é correto dizer que ele sempre reutiliza qualquer tela já existente na pilha**.
- `push` é específico de navegação em pilha e adiciona **uma nova instância** da tela.

> **Fluxos de login:** `replace` pode ser útil em alguns cenários, mas autenticação real geralmente é melhor tratada por **navegação condicional**, que oferece conjuntos de telas diferentes conforme o usuário esteja logado ou não.

## 13. Problemas comuns e como resolver

### "Cannot find module '@react-navigation/bottom-tabs'"

Instale o pacote no projeto correto:

```bash
npm install @react-navigation/bottom-tabs
```

**Cuidado:** o nome do pacote tem `@` no começo. `react-navigation/bottom-tabs` **não** é o nome correto.

### "The action 'NAVIGATE' was not handled by any navigator"

Verifique:

1. O nome da rota em `navigate('Nome')` está exatamente igual ao usado em `Stack.Screen` ou `Tab.Screen`?
2. A rota pertence àquele navegador ou está dentro de outro navegador?
3. Você configurou os tipos e parâmetros corretamente?

### "Argument of type ... is not assignable ..."

É o TypeScript indicando um possível erro nos nomes das rotas ou nos parâmetros. Confira o arquivo `types/navigation.ts` antes de tentar desativar a verificação.

### Depois de atualizar o Expo, o aplicativo não inicia

```bash
npx expo install --check
npx expo install --fix
npx expo start -c
```

**Importante:** limpar o cache não resolve todo erro. Leia o erro exibido no terminal e verifique dependências, nomes de importação e compatibilidade de versões.

### Aparecem dois cabeçalhos na aba Início

O Tab e o Stack aninhado estão desenhando cabeçalhos ao mesmo tempo. Em `AppTabs.tsx`, mantenha `headerShown: false` **na aba Inicio** para que o Stack interno assuma o cabeçalho.

---

## 14. Atividade final — Sua própria loja

A partir do exemplo da **Parte C**, personalize o aplicativo:

**Obrigatório**

- [ ] Manter **Bottom Tabs + Stack** funcionando no mesmo aplicativo.
- [ ] Criar pelo menos **cinco produtos** no arquivo `data/produtos.ts`.
- [ ] Exibir os produtos com `FlatList`.
- [ ] Navegar da lista para a tela de detalhes, enviando o `produtoId`.
- [ ] Mostrar nome, preço e descrição do produto escolhido.
- [ ] Manter as três abas: **Início, Favoritos e Configurações**.
- [ ] Utilizar **TypeScript sem `any`**.
- [ ] Manter **cada tela em um arquivo separado**.

**Desafios extras**

- [ ] Criar uma aba **Sobre** com informações do aplicativo.
- [ ] Acrescentar uma propriedade `categoria` aos produtos e exibi-la nos detalhes.
- [ ] Adicionar um botão na tela de detalhes que mostre um `Alert.alert` com o nome do produto.
- [ ] Alterar o título do cabeçalho da tela de detalhes conforme o produto selecionado.

### Perguntas para responder

1. Qual é a diferença entre Stack e Bottom Tabs?
2. Por que a tela `DetalhesScreen` não precisa ser repetida para cada produto?
3. O que significa `route.params`?
4. Para que serve `NavigatorScreenParams`?
5. Por que as abas continuam visíveis enquanto olhamos os detalhes do produto?
6. Se quiséssemos esconder as abas na tela de detalhes, o que mudaríamos na organização dos navegadores?

---

## 15. Resumo

Hoje aprendemos a criar aplicativos com múltiplas telas usando **React Navigation**:

- **Stack**: abre telas sobre outras, oferecendo retorno à tela anterior.
- **Tabs**: permite alternar entre áreas principais do aplicativo.
- **`route.params`**: recebe informações enviadas durante a navegação.
- **TypeScript**: verifica nomes de rotas e parâmetros antes da execução.
- **Navegação aninhada**: permite colocar um Stack dentro de uma aba e construir experiências mais completas.

**O conceito central:** não precisamos escolher entre Stack **ou** Tabs. Em aplicativos reais, frequentemente utilizamos **os dois juntos**.

## Referências oficiais

- [Expo SDK 58 — anúncio da versão](https://expo.dev/changelog/sdk-58)
- [Versões do Expo SDK](https://docs.expo.dev/versions/v58.0.0/)
- [React Navigation — primeiros passos](https://reactnavigation.org/docs/getting-started/)
- [React Navigation — TypeScript](https://reactnavigation.org/docs/typescript/)
- [React Navigation — navegadores aninhados](https://reactnavigation.org/docs/nesting-navigators/)
- [React Navigation — Native Stack](https://reactnavigation.org/docs/native-stack-navigator/)
- [React Navigation — Bottom Tabs](https://reactnavigation.org/docs/bottom-tab-navigator/)
