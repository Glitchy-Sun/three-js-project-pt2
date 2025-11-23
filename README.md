Um projeto interativo que utiliza **Three.js com WebGPU** para criar animações faciais em tempo real, sincronizando expressões faciais com síntese de voz usando IA local (heurística offline).

## Sobre o Projeto

Este projeto implementa um sistema de animação facial que permite:

- **Modo 1 - Letra para Expressão**: Mapear letras individuais para expressões faciais específicas
- **Modo 2 - Frase para Fala Fluida**: Converter frases em animações faciais sincronizadas com síntese de voz
- **Suporte a múltiplas vozes**: Seleção de vozes do sistema operacional
- **Calibração automática**: Ajuste automático da taxa de fala para sincronização precisa
- **Modo offline**: Funciona completamente offline sem dependências de APIs externas

---

## Tecnologias Utilizadas

### Core
- **Three.js r128** - Biblioteca 3D JavaScript
- **WebGPU** - API de renderização de alto desempenho
- **JavaScript (ES6+ Modules)**

### Recursos Three.js
- `GLTFLoader` - Carregamento de modelos 3D (formato glTF/glB)
- `KTX2Loader` - Suporte a texturas comprimidas
- `MeshoptDecoder` - Decodificação de malhas otimizadas
- `OrbitControls` - Controle interativo da câmera
- `RoomEnvironment` - Ambiente pré-configurado
- `PMREMGenerator` - Geração de mapa de radiância
- `Inspector` - Ferramenta de inspeção e debug

### APIs de Navegador
- **Web Speech API** (`SpeechSynthesis`) - Síntese de voz nativa do navegador
- **requestAnimationFrame** - Loop de animação otimizado
- **Morph Targets** - Sistema de blend shapes para deformação de malhas

### Modelo 3D
- **Face Cap Model** - Modelo facial parametrizado com suporte a morph targets
- **152+ parâmetros de blend shapes** para expressões faciais detalhadas

---

## Instalação

### Opção 1: Uso Direto (Recomendado para Desenvolvimento)

1. **Clone ou baixe o repositório**
   ```bash
   git clone <seu-repo>
   cd three-js-facial-animation
   ```

2. **Instale as dependências do Three.js**
   ```bash
   npm install three
   ```

3. **Inicie um servidor local**
   ```bash
   npm start
   ```
   
   Ou use uma alternativa simples:
   ```bash
   # Com Python 3
   python -m http.server 8000
   
   # Com Python 2
   python -m SimpleHTTPServer 8000
   
   # Com Node.js (http-server)
   npx http-server
   ```

4. **Acesse no navegador**
   ```
   http://localhost:8000/Launcher.html
   ```

### Opção 2: Setup com npm start (Recomendado)

1. **Crie um arquivo `package.json`** na raiz do projeto:
   ```json
   {
     "name": "three-js-facial-animation",
     "version": "1.0.0",
     "description": "AI Facial Animation with Three.js WebGPU",
     "main": "Launcher.html",
     "scripts": {
       "start": "npx http-server -p 8000 -c-1",
       "dev": "npx http-server -p 8000 -o Launcher.html"
     },
     "dependencies": {
       "three": "^r128"
     }
   }
   ```

2. **Instale as dependências**
   ```bash
   npm install
   ```

3. **Inicie o servidor**
   ```bash
   npm start
   ```

### Arquitetura de Pastas

```
projeto/
├── Launcher.html                    # Hub principal de seleção
├── dever_do_túlio.html              # Parte 1: Expressões básicas
├── dever_do_túlioPT1.html           # Parte 2: Com síntese de voz
├── build/
│   ├── three.webgpu.js              # Three.js com suporte WebGPU
│   └── three.tsl.js                 # Three.js Shading Language
├── jsm/
│   ├── controls/OrbitControls.js
│   ├── loaders/GLTFLoader.js
│   ├── loaders/KTX2Loader.js
│   ├── libs/meshopt_decoder.module.js
│   ├── environments/RoomEnvironment.js
│   └── inspector/Inspector.js
└── models/
    └── gltf/facecap.glb             # Modelo facial 3D
```

### Requisitos

- **Navegador moderno** com suporte a WebGPU (Chrome 113+, Edge 113+)
- **Node.js 14+** (apenas para executar servidor local)
- **Internet** para primeira carregação dos recursos (após cache, funciona offline)

---

## Como Usar

### 1. Iniciar a Aplicação

Abra `Launcher.html` em seu navegador. Você verá dois projetos disponíveis:
- **Projeto 1**: Animação básica (Letra → Expressão)
- **Projeto 2**: Animação avançada (Frase → Fala + Expressão sincronizada)

### 2. Modo Letra → Expressão

1. Digite uma ou mais letras (ex: `A, B, C` ou `A e B e C`)
2. Clique em **"Gerar Expressão"**
3. A face animará com as expressões correspondentes

### 3. Modo Frase → Fala Sincronizada

1. Digite uma frase completa no campo de texto
2. **Opcional**: Selecione uma voz diferente no dropdown
3. Clique em **"Falar Frase"**
4. A face animará sincronizada com a síntese de voz

**Recursos avançados:**
- **Calibração de Voz**: Ajusta automaticamente o tempo de sincronização
- **Fluidez da Face**: Controle de suavidade da animação (0.50 - 1.00)
- **Taxa de Fala**: Multiplicador de velocidade

---

## ⚙️ Configuração Técnica

### Import Map (Resolução de Módulos)

```javascript
{
  "imports": {
    "three": "../build/three.webgpu.js",
    "three/webgpu": "../build/three.webgpu.js",
    "three/tsl": "../build/three.tsl.js",
    "three/addons/": "./jsm/"
  }
}
```

### Principais Componentes

**LocalFaceAI**: Gerador heurístico de expressões offline
- Não requer API externa
- Usa regras simples de mapeamento
- Suporta síntese de sílabas

**Morph Targets**: 152+ parâmetros de deformação facial
- `jawOpen`, `mouthOpen`, `mouthPucker`
- `eyeSquint_L/R`, `eyeWide_L/R`
- `mouthSmile_L/R`, `browInnerUp`
- E muitos mais...

---

## Principais Desafios e Soluções

### 1. **Sincronização Face ↔ Fala**

#### Problema
A maior dificuldade foi sincronizar a animação facial com a síntese de voz nativa do navegador (Web Speech API). Os principais obstáculos foram:

- **Variação de duração**: Cada navegador/SO sintetiza voz em tempo diferente
- **Imprecisão do timing**: `speechSynthesis` não fornece callbacks precisos de progresso
- **Latência**: Delay entre geração de keyframes e reprodução de áudio

#### Solução Implementada

1. **Calibração Automática**: Sistema de auto-calibração que:
   - Mede a duração real de uma frase de teste
   - Compara com duração estimada (0.06s por caractere)
   - Calcula fator de correção (`speechRateCalibration`)

2. **Keyframes Baseados em Sílabas**: 
   - Análise de sílabas para timing mais natural
   - Geração de frames de "fechamento" para consoantes
   - Pausas explícitas para pontuação

3. **Ajuste de Taxa de Fala**:
   ```javascript
   let desiredRate = (estimatedDuration * calibration) / totalAnimDuration;
   desiredRate = Math.max(0.5, Math.min(2.0, desiredRate * 1.10));
   ```

4. **Fluidez Dinâmica** (`faceFluency`):
   - Controle de suavidade (0.50 a 1.00)
   - Afeta tanto duração quanto intensidade da animação

### 2. **Descontinuação de API Externa**

#### Problema
O projeto foi inicialmente concebido com integração a uma API de IA remota para:
- Análise de linguagem natural
- Sincronização inteligente
- Previsão de visemas

**Esta API foi descontinuada**, deixando a aplicação quebrada.

#### Solução Implementada

**Reimplementação com IA Local (Heurística)**:

```javascript
class LocalFaceAI {
  generateExpressionForLetter(letter) {
    // Mapeamento simples mas eficaz
    const mood = { 'A': 'HAPPY', 'O': 'SURPRISED', ... }
    return generateMorphTargets(mood)
  }
  
  generateSyllableKeyframesForText(text) {
    // Análise de sílabas para timing natural
    // Sem dependência de API externa
  }
}
```

**Vantagens da solução offline**:
- ✅ Sem custos de API
- ✅ Sem latência de rede
- ✅ Funciona completamente offline
- ✅ Maior privacidade (dados locais)
- ✅ Totalmente controlável e customizável

**Trade-offs aceitáveis**:
- Expressões menos sofisticadas (mas ainda convincentes)
- Sincronização aproximada (mas adequada para uso interativo)

### 3. **Compatibilidade WebGPU**

#### Problema
WebGPU é ainda experimental/recente em muitos navegadores

#### Solução
- Suporte limitado a navegadores modernos (Chrome 113+, Edge 113+)
- Fallback potencial para WebGL em versões futuras
- Clear error messages se WebGPU não estiver disponível

---

## Segurança & Privacidade

- ✅ Sem envio de dados para servidor
- ✅ Sem tracking
- ✅ Funciona em rede local
- ✅ Completamente offline após cache

---

## Troubleshooting

**Problema**: "WebGPU não disponível"
- **Solução**: Use navegador atualizado (Chrome/Edge versão 113+)

**Problema**: Modelo 3D não carrega
- **Solução**: Verifique se caminho `models/gltf/facecap.glb` está correto

**Problema**: Áudio da síntese não toca
- **Solução**: Verifique permissões de áudio do navegador; teste em HTTPS

**Problema**: Animação descascada com áudio
- **Solução**: Clique em "Auto-calibrar" para medir sincronização correta

---

## Recursos

- [Three.js Documentation](https://threejs.org/docs/)
- [WebGPU Specification](https://www.w3.org/TR/webgpu/)
- [Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API)

- [glTF Format](https://www.khronos.org/gltf/)

---

## Licença

Este projeto utiliza recursos de [Three.js](https://github.com/mrdoob/three.js) (MIT License) e modelo de [Face Cap](https://www.bannaflak.com/face-cap).

## Imagens

<img width="1853" height="893" alt="Captura de tela 2025-11-23 152907" src="https://github.com/user-attachments/assets/43b0d88f-3c86-4bfc-bc8c-4df8363dc728" />
<img width="1698" height="961" alt="Captura de tela 2025-11-23 152829" src="https://github.com/user-attachments/assets/8d0ab073-50e8-470a-a8cf-636a23d59793" />
<img width="1904" height="863" alt="Captura de tela 2025-11-23 152747" src="https://github.com/user-attachments/assets/a17f68c6-f99c-4421-ad91-ffd492a7475e" />
