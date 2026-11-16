# **RN**

# UC15 · Interface Gráfica Mobile

**Técnico em Desenvolvimento de Sistemas**  
React Native · Expo SDK 58 · TypeScript · Android/iOS/Web

---

**Bloco 2 · Aula 11**

## Geolocalização: pegando a posição do usuário e mostrando num mapa

Crie **um projeto novo** para esta aula. Hoje juntamos três peças:

1. **`expo-location`**: pergunta ao sistema operacional onde o usuário está e devolve coordenadas.
2. **`react-native-maps`**: desenha mapas no **Android e iOS**, com marcadores e gestos.
3. **`react-leaflet` + `leaflet`**: desenha mapas **no navegador**, usando dados do OpenStreetMap.

O objetivo é ter **um único aplicativo Expo** que funcione tanto no **celular** quanto na **Web**, compartilhando a lógica de localização.

> **O que mudou no Expo 58?** O SDK 58 utiliza React Native 0.88. As funções tradicionais de `expo-location` continuam válidas, assim como `MapView` e `Marker` de `react-native-maps` no mobile. A principal adaptação para esta aula é criar um mapa específico para Web, porque `react-native-maps` não fornece o mesmo componente para navegador.

### 1. Três peças, três bibliotecas

| Biblioteca | Responsabilidade | Plataformas nesta aula |
|---|---|---|
| `expo-location` | Permissão e coordenadas do usuário | Android, iOS e Web* |
| `react-native-maps` | Mapa nativo, pin e zoom | Android e iOS |
| `react-leaflet` + `leaflet` | Mapa interativo no navegador | Web |

\* A localização na Web depende da permissão do navegador e de um contexto seguro, geralmente **HTTPS** ou **`localhost`**.

**Terminal — criando um projeto específico para o SDK 58**

```bash
npx create-expo-app@latest mapa-localizacao --template blank-typescript@sdk-58
cd mapa-localizacao
```

> **Atenção:** não dependa apenas de `create-expo-app@latest` sem especificar o SDK; durante o período de transição, ele pode gerar uma versão anterior. Confira o `package.json`: a dependência `expo` deve estar na família **`~58.x`**. Se o template versionado não estiver disponível no momento, consulte o guia oficial do SDK 58 indicado no final da aula.

**Terminal — instalando bibliotecas do aplicativo**

```bash
npx expo install expo-location react-native-maps
npx expo install react-dom react-native-web @expo/metro-runtime
npm install leaflet react-leaflet
npm install -D @types/leaflet
npx expo install --check
```

**Dica:** `npx expo install` escolhe versões adequadas ao SDK do projeto. Já `npm install` é usado aqui para as bibliotecas específicas do navegador.

**Estrutura que vamos criar**

```text
mapa-localizacao/
├── components/
│   ├── MapaLugares.tsx          # Android e iOS
│   └── MapaLugares.web.tsx      # Navegador
├── App.tsx                       # Lógica compartilhada
├── app.json
├── package.json
└── tsconfig.json
```

**Por que dois arquivos com quase o mesmo nome?** O Expo utiliza extensões por plataforma. Na Web, ele prioriza `MapaLugares.web.tsx`. No celular, utiliza `MapaLugares.tsx`. O `App.tsx` importa sempre o mesmo caminho e não precisa saber qual mapa será desenhado.

> **Dica — Expo Go:** a documentação do Expo inclui `react-native-maps` no Expo Go, sem exigir uma chave adicional do Google para esse teste. A publicação em lojas pode exigir configuração adicional. Verifique se o **Expo Go instalado no celular suporta o SDK 58**; caso ainda não suporte, será necessário atualizar o aplicativo ou utilizar uma development build compatível.

### 2. Pedindo permissão de localização

Um aplicativo **não deve acessar a localização sem permissão**. O sistema operacional ou navegador apresenta uma confirmação ao usuário.

**Trecho — `expo-location`**

```tsx
import * as Location from 'expo-location';

const { status } =
  await Location.requestForegroundPermissionsAsync();

if (status !== 'granted') {
  Alert.alert('Permissão necessária', 'Precisamos da localização.');
  return;
}
```

> **Atenção:** `Alert.alert()` é próprio do ambiente nativo. Como nosso aplicativo também roda na Web, no exemplo completo usaremos um estado de erro e um componente `Text`, que funcionam nas duas plataformas.

**O que acontece aqui?**

- `requestForegroundPermissionsAsync()` solicita permissão de localização **enquanto o aplicativo está em uso**.
- `status === 'granted'` indica que a permissão foi concedida.
- Se a permissão for negada, o aplicativo deve mostrar uma mensagem compreensível, sem ficar carregando para sempre.

Existe também `requestBackgroundPermissionsAsync()`, para casos especiais de localização em segundo plano. **Não vamos utilizá-lo nesta aula**, porque exige cuidados adicionais com privacidade, plataforma e configuração.

### 3. Pegando a posição atual

Após receber permissão, podemos solicitar a localização atual **uma única vez**.

**Trecho**

```tsx
const posicao = await Location.getCurrentPositionAsync({
  accuracy: Location.Accuracy.Balanced,
});

console.log('Latitude:', posicao.coords.latitude);
console.log('Longitude:', posicao.coords.longitude);
```

O retorno contém, entre outros dados:

```ts
{
  coords: {
    latitude: number,
    longitude: number,
    accuracy: number | null,
    altitude: number | null,
    speed: number | null,
    // Existem outras propriedades, conforme a plataforma.
  },
  timestamp: number,
}
```

- **Latitude:** posição no eixo norte-sul.
- **Longitude:** posição no eixo leste-oeste.
- **Accuracy:** precisão estimada da localização, em metros, quando disponível.
- **Timestamp:** momento da leitura.

**Exemplo:** um ponto pode ser representado por `{ latitude: -29.7600, longitude: -51.1500 }`. Essas coordenadas são apenas ilustrativas.

> **Dica:** GPS, Wi-Fi e rede móvel podem influenciar a precisão. O emulador pode utilizar uma posição simulada, e o navegador pode estimar a localização de forma diferente do celular.

### 4. Criando o `App.tsx`: lógica compartilhada

Vamos deixar a permissão, a obtenção das coordenadas e os estados de carregamento/erro no **`App.tsx`**. Assim, cada mapa recebe somente **latitude** e **longitude**.

**`App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import {
  View,
  Text,
  ActivityIndicator,
  StyleSheet,
  Pressable,
} from 'react-native';
import * as Location from 'expo-location';

import MapaLugares from './components/MapaLugares';

export default function App() {
  const [localizacao, setLocalizacao] =
    useState<Location.LocationObjectCoords | null>(null);
  const [erro, setErro] = useState('');
  const [carregando, setCarregando] = useState(true);

  const obterLocalizacao = async () => {
    setCarregando(true);
    setErro('');

    try {
      const { status } =
        await Location.requestForegroundPermissionsAsync();

      if (status !== 'granted') {
        setErro('Permissão de localização negada.');
        return;
      }

      const posicao = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.Balanced,
      });

      setLocalizacao(posicao.coords);
    } catch (error) {
      console.error(error);
      setErro('Não foi possível obter a localização.');
    } finally {
      setCarregando(false);
    }
  };

  useEffect(() => {
    obterLocalizacao();
  }, []);

  return (
    <View style={styles.container}>
      <View style={styles.cabecalho}>
        <Text style={styles.titulo}>Meu Mapa de Favoritos</Text>
        <Pressable style={styles.botao} onPress={obterLocalizacao}>
          <Text style={styles.textoBotao}>Atualizar posição</Text>
        </Pressable>
      </View>

      {carregando ? (
        <View style={styles.centro}>
          <ActivityIndicator size="large" />
          <Text>Obtendo sua localização...</Text>
        </View>
      ) : erro ? (
        <View style={styles.centro}>
          <Text>{erro}</Text>
          <Text>Tente permitir a localização e atualizar.</Text>
        </View>
      ) : localizacao ? (
        <MapaLugares
          key={`${localizacao.latitude}-${localizacao.longitude}`}
          latitude={localizacao.latitude}
          longitude={localizacao.longitude}
        />
      ) : null}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  cabecalho: { padding: 16, gap: 10, backgroundColor: '#f1f5f9' },
  titulo: { fontSize: 20, fontWeight: 'bold' },
  botao: {
    alignSelf: 'flex-start',
    backgroundColor: '#2563eb',
    paddingVertical: 10,
    paddingHorizontal: 14,
    borderRadius: 8,
  },
  textoBotao: { color: '#fff', fontWeight: 'bold' },
  centro: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    gap: 12,
    padding: 20,
  },
});
```

**Observe:**

- `useState` armazena a localização, erros e carregamento.
- `useEffect` inicia a localização ao abrir o aplicativo.
- `try/catch/finally` trata problemas sem travar a tela em carregamento.
- `Pressable` oferece um botão para atualizar a posição.
- `key` força a recriação do mapa ao atualizar a localização. É uma solução simples para esta prática porque os mapas usam uma região inicial; em apps mais avançados, é possível controlar a câmera sem recriar o componente.

> **Nota:** para simplificar o primeiro exemplo, a operação assíncrona de localização foi colocada diretamente no `App.tsx`. Num projeto maior, é recomendado proteger atualizações de estado após a desmontagem do componente ou extrair essa lógica para um hook próprio.

### 5. Mostrando um mapa com pin no Android e iOS

No mobile, usamos `MapView` para mostrar o mapa e `Marker` para adicionar marcadores.

**`components/MapaLugares.tsx` — Android e iOS**

```tsx
import { StyleSheet } from 'react-native';
import MapView, { Marker } from 'react-native-maps';

interface Props {
  latitude: number;
  longitude: number;
}

interface Lugar {
  id: string;
  nome: string;
  latitude: number;
  longitude: number;
}

export default function MapaLugares({ latitude, longitude }: Props) {
  const lugares: Lugar[] = [
    {
      id: '1',
      nome: 'Lugar favorito 1',
      latitude: latitude + 0.003,
      longitude: longitude + 0.003,
    },
    {
      id: '2',
      nome: 'Lugar favorito 2',
      latitude: latitude - 0.004,
      longitude: longitude + 0.002,
    },
    {
      id: '3',
      nome: 'Lugar favorito 3',
      latitude: latitude + 0.002,
      longitude: longitude - 0.004,
    },
  ];

  return (
    <MapView
      style={styles.mapa}
      initialRegion={{
        latitude,
        longitude,
        latitudeDelta: 0.03,
        longitudeDelta: 0.03,
      }}
      showsUserLocation={true}
    >
      {lugares.map((lugar) => (
        <Marker
          key={lugar.id}
          coordinate={{
            latitude: lugar.latitude,
            longitude: lugar.longitude,
          }}
          title={lugar.nome}
          description="Um lugar favorito perto de você"
        />
      ))}
    </MapView>
  );
}

const styles = StyleSheet.create({
  mapa: { flex: 1 },
});
```

**Mapa centralizado na posição inicial do usuário, com três pins e o ponto azul do sistema.**

> **Dica:** `latitudeDelta` e `longitudeDelta` determinam a área visível aproximada. Valores menores exibem uma área mais próxima. `initialRegion` define a região **inicial**, mas não reposiciona automaticamente o mapa em toda atualização posterior.

### 6. Mostrando o mesmo mapa na Web

No navegador, o equivalente é o **React Leaflet**, construído sobre o **Leaflet**. Usaremos os tiles do **OpenStreetMap**, com atribuição obrigatória ao projeto.

**`components/MapaLugares.web.tsx` — Web**

```tsx
import {
  MapContainer,
  TileLayer,
  Marker,
  Popup,
  CircleMarker,
} from 'react-leaflet';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

interface Props {
  latitude: number;
  longitude: number;
}

interface Lugar {
  id: string;
  nome: string;
  latitude: number;
  longitude: number;
}

const icone = L.divIcon({
  html: '<span style="font-size:30px">📍</span>',
  className: '',
  iconSize: [32, 40],
  iconAnchor: [16, 40],
});

export default function MapaLugares({ latitude, longitude }: Props) {
  const lugares: Lugar[] = [
    {
      id: '1',
      nome: 'Lugar favorito 1',
      latitude: latitude + 0.003,
      longitude: longitude + 0.003,
    },
    {
      id: '2',
      nome: 'Lugar favorito 2',
      latitude: latitude - 0.004,
      longitude: longitude + 0.002,
    },
    {
      id: '3',
      nome: 'Lugar favorito 3',
      latitude: latitude + 0.002,
      longitude: longitude - 0.004,
    },
  ];

  return (
    <MapContainer
      center={[latitude, longitude]}
      zoom={14}
      style={{ width: '100%', height: '100%' }}
    >
      <TileLayer
        attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      />

      <CircleMarker
        center={[latitude, longitude]}
        radius={9}
        pathOptions={{
          color: '#fff',
          fillColor: '#1683ff',
          fillOpacity: 1,
          weight: 3,
        }}
      >
        <Popup>Você está aqui!</Popup>
      </CircleMarker>

      {lugares.map((lugar) => (
        <Marker
          key={lugar.id}
          position={[lugar.latitude, lugar.longitude]}
          icon={icone}
        >
          <Popup>{lugar.nome}</Popup>
        </Marker>
      ))}
    </MapContainer>
  );
}
```

**O que muda de uma plataforma para outra?**

| No celular (`react-native-maps`) | Na Web (`react-leaflet`) |
|---|---|
| `<MapView>` | `<MapContainer>` |
| `initialRegion` | `center` e `zoom` |
| `<Marker coordinate={{ latitude, longitude }}>` | `<Marker position={[latitude, longitude]}>` |
| `title` do marcador | `<Popup>` |
| `showsUserLocation` | Marcador azul criado manualmente |

**Atenção:** o ponto azul usado com `CircleMarker` é um marcador desenhado com as **coordenadas obtidas naquele momento**. Ele não rastreia o movimento automaticamente. Para isso, precisamos da função da próxima seção.

**Observação para projetos reais:** o servidor público de tiles do OpenStreetMap possui política de uso e não é uma solução garantida para aplicações de alto tráfego. É adequado conhecer o serviço em uma atividade pequena, respeitando a atribuição; em produção, avalie um provedor de mapas apropriado.

### 7. Executando no celular e no navegador

**Web — terminal**

```bash
npx expo start --web
```

Abra o endereço apresentado no terminal (normalmente `http://localhost:8081`). Quando o navegador solicitar localização, clique em **Permitir**.

**Android/iOS — Expo Go**

```bash
npx expo start
```

Abra o **Expo Go compatível com o SDK 58** e escaneie o QR Code. Computador e celular devem conseguir acessar um ao outro na rede.

**Se a rede impedir a conexão local:**

```bash
npx expo start --tunnel
```

> **Atenção — Web e segurança:** a geolocalização do navegador funciona normalmente em **`localhost`** ou **HTTPS**. Abrir `http://192.168.x.x:8081` em outro aparelho pode impedir o acesso à localização por não ser um contexto seguro. No celular, para este exercício, prefira **Expo Go**, e no computador use **localhost**.

**Checklist de teste:**

- [ ] O aplicativo abre no navegador.
- [ ] O navegador pede permissão.
- [ ] O mapa mostra a posição e três marcadores.
- [ ] O aplicativo abre no celular.
- [ ] O ponto azul da localização aparece no mapa nativo.
- [ ] Clicar/tocar num lugar favorito mostra o nome.
- [ ] Negar a localização exibe uma mensagem em vez de carregamento infinito.
- [ ] O botão **Atualizar posição** realiza outra leitura.

### 8. Bônus: observando a posição mudar em tempo real

`getCurrentPositionAsync()` solicita **uma leitura**. Para acompanhar mudanças enquanto o aplicativo está em uso, utilizamos `watchPositionAsync()`.

**Bônus — exemplo de `useEffect` com cleanup**

O trecho abaixo é **alternativo** ao efeito de leitura única: ele demonstra um observador e não deve ser simplesmente colado como um segundo `useEffect` no projeto final sem adaptar os estados. Considere que `setLocalizacao` já existe.

```tsx
useEffect(() => {
  let ativo = true;
  let inscricao: Location.LocationSubscription | null = null;

  const iniciar = async () => {
    try {
      const { status } =
        await Location.requestForegroundPermissionsAsync();

      if (status !== 'granted' || !ativo) return;

      const novaInscricao = await Location.watchPositionAsync(
        {
          accuracy: Location.Accuracy.Balanced,
          distanceInterval: 10,
          timeInterval: 5000,
        },
        (posicao) => {
          if (ativo) {
            setLocalizacao(posicao.coords);
          }
        }
      );

      if (ativo) {
        inscricao = novaInscricao;
      } else {
        novaInscricao.remove();
      }
    } catch (error) {
      console.error('Falha ao acompanhar localização:', error);
    }
  };

  iniciar();

  return () => {
    ativo = false;
    inscricao?.remove();
  };
}, []);
```

**Por que cancelar?** O `watchPositionAsync()` cria uma inscrição. Ao sair da tela, o componente deve chamar `remove()` para não manter observadores desnecessários. Como a inscrição é obtida de forma assíncrona, também precisamos cuidar do caso em que o componente é desmontado **antes de a inscrição ficar pronta**.

> **Importante:** `timeInterval` tem comportamento específico no Android. A frequência real depende da plataforma, das permissões e das condições do dispositivo. Na Web, o navegador pode limitar ou aproximar as atualizações. O GPS também pode não mudar se o aluno estiver parado.

### 9. Referência rápida

| API / prop | Biblioteca | Para que serve |
|---|---|---|
| `requestForegroundPermissionsAsync()` | `expo-location` | Pede permissão para localização em primeiro plano |
| `getCurrentPositionAsync(opções)` | `expo-location` | Obtém a localização uma vez |
| `watchPositionAsync(opções, callback)` | `expo-location` | Observa mudanças de posição; precisa de cleanup |
| `Location.Accuracy.Balanced` | `expo-location` | Solicita precisão equilibrada |
| `initialRegion` | `MapView` | Centro e zoom inicial do mapa nativo |
| `showsUserLocation` | `MapView` | Exibe o indicador padrão da posição do usuário no mapa nativo |
| `coordinate` | `Marker` nativo | Define a coordenada do pin como objeto |
| `title` / `description` | `Marker` nativo | Texto apresentado ao tocar no marcador |
| `center` / `zoom` | `MapContainer` | Define centro e zoom inicial na Web |
| `position` | `Marker` do Leaflet | Define coordenada como array `[latitude, longitude]` |
| `TileLayer` | React Leaflet | Desenha as imagens que compõem o mapa |
| `Popup` | React Leaflet | Exibe informação sobre um marcador |
| `.web.tsx` | Expo / Metro | Escolhe automaticamente um componente para a Web |

### 10. Prática

> ## Mão na massa — Meu Mapa de Favoritos
>
> Crie um aplicativo Expo SDK 58 que funcione **no celular e no navegador**. O projeto deve solicitar permissão de localização e mostrar um mapa centralizado na posição do usuário. Adicione **três lugares favoritos** com nome e coordenadas próximas à localização recebida; cada um deve ter seu próprio marcador.

**Requisitos obrigatórios**

1. Criar o projeto com **React Native + Expo 58 + TypeScript**.
2. Pedir permissão de localização **em primeiro plano**.
3. Mostrar uma mensagem se a permissão for negada ou a posição não puder ser obtida.
4. Exibir a localização do usuário no mapa.
5. Criar um **array com três lugares favoritos** (`id`, `nome`, `latitude` e `longitude`).
6. Renderizar um marcador para cada lugar usando `.map()`.
7. Mostrar o nome do lugar ao **tocar no celular** ou **clicar na Web**.
8. Adicionar um botão **Atualizar posição**.
9. Executar e testar o aplicativo **nas duas plataformas**.

**Bônus (opcional):** substituir a consulta única por acompanhamento em tempo real com `watchPositionAsync()`, usando cleanup correto.

**Entrega:** repositório GitHub contendo o código-fonte e **duas capturas de tela**, uma do navegador e outra do celular. Descreva no `README.md` como executar o aplicativo.

---

### 11. Problemas comuns

| Problema | O que conferir |
|---|---|
| O projeto inicia com SDK 57 | Confira a versão do `expo` no `package.json`; crie com template específico do SDK 58 |
| O Expo Go não abre o projeto | Confira se o Expo Go instalado é compatível com o SDK usado |
| Mapa funciona no celular, mas quebra na Web | Verifique se `MapaLugares.web.tsx` existe e importa somente `react-leaflet` |
| O mapa está em branco | Confirme que a tela tem altura (`flex: 1` no container; `height: '100%'` no mapa web) |
| O navegador não obtém localização | Teste em `localhost` ou HTTPS e verifique a permissão do site |
| `react-leaflet` reclama de tipo ou estilo | Confirme `leaflet`, `react-leaflet`, `@types/leaflet` e a importação de `leaflet.css` |
| O botão atualiza coordenadas, mas o centro não muda | Lembre que `initialRegion` e `center` são iniciais; neste exercício a `key` recria o mapa |
| O TypeScript mostra erros de versões | Rode `npx expo install --check` e `npx tsc --noEmit` |

