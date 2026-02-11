# Revisão técnica — Atlas v1.7 Cubo Integrado

## O que esse código é

Este HTML cria uma visualização 3D interativa com **Three.js** contendo:

- Um **CORE** central (esfera magenta).
- Um **cubo wireframe** translúcido ao redor.
- 4 nós **PRIMARY** e 4 nós **MEDIUM** com cores base.
- 2 nós **externos** (`E1`, `E2`).
- **Conexões** em linha entre nós (core/primary/medium/externos).
- **OrbitControls** para rotação com mouse.
- **Raycasting de hover** para destacar conexões.
- **Partículas de energia** (pulsos) que percorrem as conexões quando há hover.

---

## O que está bom

- Estrutura geral clara e organizada por blocos (`SCENE`, `CAMERA`, `LIGHT`, etc.).
- Separação razoável por funções (`createLabel`, `createConnection`, `createExternalNode`, `createPulse`).
- Interação de hover funcionando de forma direta.
- Resize responsivo da câmera/renderizador.

---

## O que precisa ajustar (prioridade alta)

1. **Acúmulo excessivo de partículas (risco de queda de FPS)**
   - `createPulse` é chamado sempre que há hover, em todo frame, para cada conexão relevante.
   - Em 60 FPS, isso pode gerar centenas de meshes rapidamente.
   - **Ajuste recomendado:** adicionar _throttle_ por conexão (ex.: 1 pulso a cada 120–200ms) ou limite global de partículas.

2. **Remoção de partículas usando `splice` dentro de `forEach`**
   - Modificar array durante `forEach` pode pular elementos.
   - **Ajuste recomendado:** iterar de trás para frente com `for (let i = arr.length - 1; i >= 0; i--)`.

3. **Possível vazamento de memória em recursos WebGL**
   - Ao remover pulso da cena, geometria/material não são descartados.
   - **Ajuste recomendado:** chamar `geometry.dispose()` e `material.dispose()` quando remover cada pulso.

4. **Conexões não acompanham nós se posições mudarem no futuro**
   - `BufferGeometry` das linhas é criada com pontos fixos.
   - Atualmente os nós não se movem, então ok, mas limita evolução.
   - **Ajuste recomendado:** guardar referência de atributos de posição e atualizar no loop se houver animação de nós.

---

## Melhorias recomendadas (média prioridade)

1. **Compatibilidade de versão do Three.js**
   - CDN fixa em `0.128.0` (antiga).
   - **Ajuste recomendado:** atualizar para versão recente e migrar `OrbitControls` para módulo ES (`three/addons/...`) com build por bundler ou importmap.

2. **Qualidade de texto dos labels**
   - Canvas fixo pode ficar borrado em telas high-DPI.
   - **Ajuste recomendado:** usar `window.devicePixelRatio` para dimensionar o canvas e manter nitidez.

3. **Legibilidade em fundo escuro**
   - Labels e linhas com opacidade baixa podem ficar fracos.
   - **Ajuste recomendado:** aplicar contorno/sombra no texto e elevar opacidade mínima das linhas (ex.: 0.35).

4. **Interação em dispositivos touch**
   - Hover por `mousemove` não existe em touch.
   - **Ajuste recomendado:** suporte a `pointermove`/`pointerdown` e seleção por toque.

5. **Controle de zoom/pan**
   - Pan desativado, zoom permitido sem limite.
   - **Ajuste recomendado:** definir `controls.minDistance` e `controls.maxDistance` para UX previsível.

---

## Melhorias opcionais (baixa prioridade)

- Adicionar `AmbientLight` suave para melhor profundidade visual.
- Nomear conexões por tipo (`core-primary`, `medium-primary`, etc.) para debugging.
- Modularizar em classes/arquivos (`GraphScene`, `InteractionController`, etc.) se o projeto crescer.
- Exibir tooltip/overlay HTML com metadados do nó ativo.

---

## Exemplo rápido de correção crítica (pulsos)

Ideia resumida:

- Incluir `lastPulseAt` em cada conexão.
- No `highlightConnections`, só criar novo pulso se `Date.now() - lastPulseAt > 150`.
- No loop de partículas, remover de trás para frente e fazer `dispose` de material/geometria.

Isso sozinho já tende a melhorar bastante a performance e estabilidade.
