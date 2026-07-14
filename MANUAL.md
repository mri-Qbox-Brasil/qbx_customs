# qbx_customs — Manual

Oficina de customização de veículos por polyzone: performance, peças, pintura (incluindo Chameleon), neon, rodas e reparo, com câmera orbital e cobrança configurável. Fork do `popcornrp-customs`.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Zonas](#zonas)
5. [Preços e cobrança](#preços-e-cobrança)
6. [Menus](#menus)
7. [Persistência](#persistência)
8. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
9. [Localização](#localização)
10. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `qbx_core` | Sim | `GetPlayer`, dinheiro, job, notificações, `playerdata`, `lib` |
| `ox_lib` | Sim | Menus, polyzones, callbacks, `getVehicleProperties`, locale, `versionCheck` |
| `oxmysql` | Sim | Salva os mods na tabela `player_vehicles` |

---

## Instalação

1. Copie a pasta `qbx_customs` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure qbx_customs
   ```
3. **Configure as zonas** em `config/shared.lua`. As coordenadas que vêm de fábrica são de mapas específicos (MRPD e Benny's do Molo) e precisam ser ajustadas para o seu servidor.
4. A tabela `player_vehicles` (padrão do `qbx_core`) precisa existir. Não há SQL próprio.

**Conflitos** — substitui qualquer outro recurso de LS Customs / Benny's (`qb-customs`, `bennys`, etc.). Rodar dois ao mesmo tempo gera zonas e menus duplicados.

O recurso registra `carcols_gen9.meta` e `carmodcols_gen9.meta` como data files, além de streamar `vehicle_paint_ramps.ytd` — necessários para as pinturas Chameleon.

---

## Configuração

### `config/shared.lua`

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `debug` | bool | Sim | Desenha as polyzones em modo debug |
| `zones` | ZoneOptions[] | Sim | Lista de oficinas. Ver [Zonas](#zonas) |
| `prices.cosmetic` | number | Sim | Preço fixo de qualquer mod cosmético (peças, extras). Padrão: `500` |
| `prices.colors` | number | Sim | Preço fixo de qualquer alteração de cor/pintura/neon. Padrão: `1000` |
| `prices[11]` | number[] | Sim | Preço do motor por nível. Padrão: `{0, 10000, 20000, 30000, 40000}` |
| `prices[12]` | number[] | Sim | Preço dos freios por nível. Padrão: `{0, 2500, 5000, 7500}` |
| `prices[13]` | number[] | Sim | Preço da transmissão por nível. Padrão: `{0, 5000, 10000, 15000, 20000}` |
| `prices[15]` | number[] | Sim | Preço da suspensão por nível. Padrão: `{0, 3000, 6000, 9000, 12000, 15000}` |
| `prices[18]` | number | Sim | Preço fixo do turbo. Padrão: `10000` |

### `config/client.lua`

Basicamente um catálogo de labels. Alterar aqui muda apenas o que aparece no menu — os IDs são os nativos do GTA V.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `currency` | string | Sim | Prefixo monetário exibido nos menus. Padrão: `R$` |
| `mods` | array | Sim | Lista de mods disponíveis: `{ id, label, category, enabled? }`. `category` é `parts` ou `performance`. `enabled = false` esconde o mod (blindagem `16` e nitro `17` vêm desligados) |
| `paints` | table | Sim | Pinturas agrupadas em `Classic`, `Matte`, `Metal`, `Worn` e `Chameleon`, cada uma com `{ id, label }` |
| `modLabels` | table | Sim | Labels dos níveis por tipo de mod (`11` motor, `12` freios, `13` transmissão, `14` buzinas, `15` suspensão, `16` blindagem) |
| `xenon` | array | Sim | Cores de farol de xênon |
| `windowTints` | array | Sim | Películas de vidro |
| `plateIndexes` | array | Sim | Modelos de placa |
| `neon` | array | Sim | Posições de neon (esquerda, direita, frente, traseira) |
| `neonColors` | array | Sim | Cores de neon (`label`, `r`, `g`, `b`) |
| `tyreSmoke` | array | Sim | Cores de fumaça de pneu (`label`, `r`, `g`, `b`) |
| `wheels` | array | Sim | Tipos de roda do GTA V |

---

## Zonas

Cada item de `zones` é uma polyzone definida por `points`. Todos os pontos devem ter **o mesmo valor de Z** — alturas diferentes quebram a zona.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `points` | vec3[] | Sim | Vértices da polyzone |
| `hideBlip` | bool | Não | Se `true`, não cria blip. Se `false`/ausente, `blip` precisa estar definido |
| `blip` | table | Não | `sprite` (padrão 72), `color` (padrão 4), `scale` (padrão 0.8), `label` (padrão `Customs`). O blip fica no centro geométrico dos pontos |
| `job` | string[] | Não | Restringe a oficina a esses jobs. O jogador precisa estar **em serviço** (`onduty`) |
| `freeRepair` | string[] | Não | Jobs que reparam de graça nessa zona |
| `freeMods` | string[] | Não | Jobs que instalam mods de graça nessa zona |
| `allowedClasses` | table<number, boolean> | Não | Somente essas classes de veículo entram |
| `deniedClasses` | table<number, boolean> | Não | Essas classes de veículo são bloqueadas |
| `modelBlacklist` | table<number, boolean> | Não | Hashes de modelo bloqueados |

O jogador precisa estar **dentro do veículo** e dentro da zona. Um TextUI aparece à direita; pressionar `E` (controle 38) zera a velocidade do carro, abre o menu e liga a câmera orbital.

As quatro zonas que vêm de fábrica (todas com `hideBlip = true`): LS Customs do centro, Hawick/La Mesa, Grapeseed e Paleto.

---

## Preços e cobrança

O cliente pede o pagamento ao servidor antes de aplicar cada modificação. O servidor:

1. Consulta a zona atual do jogador (callback `qbx_customs:client:zone`).
2. Se o job do jogador está em `freeMods` (ou `freeRepair`, no caso do reparo) daquela zona, libera sem cobrar.
3. Caso contrário, tenta debitar de `cash`; se não houver saldo, debita de `bank` e notifica. Sem saldo em nenhum dos dois, a modificação é recusada.

**Reparo** — o preço é calculado dinamicamente: `ceil(1000 - bodyHealth)`. Um carro intacto (1000 de body health) não mostra a opção de reparo; um destruído custa até $1000. O reparo restaura body e engine health, mantendo o nível de combustível.

Quando o veículo está danificado, o menu principal exibe **apenas** a opção de reparo — os menus de customização só liberam com o carro inteiro.

---

## Menus

| Menu | Conteúdo |
|---|---|
| Principal | Reparo (se danificado) ou Performance / Peças / Cores / Extras |
| Performance | Motor, freios, transmissão, suspensão, turbo. Preço por nível |
| Peças | Todos os mods de `category = 'parts'` mais rodas |
| Cores | Pintura primária/secundária, perolizado, neon, xênon, película, placa, fumaça de pneu |
| Extras | Só aparece se o veículo tiver extras (`DoesExtraExist`) |

Durante o menu, os controles de acelerar, frear, virar as rodas e trocar de rádio ficam desabilitados. A câmera orbital (`dragcam`) é controlada pelo mouse, com zoom via scroll.

---

## Persistência

Ao fechar o menu, o cliente dispara `qbx_customs:server:saveVehicleProps`. O servidor lê as propriedades do veículo, verifica se a placa existe em `player_vehicles` e, em caso positivo, atualiza a coluna `mods`:

```sql
UPDATE player_vehicles SET mods = ? WHERE plate = ?
```

Veículos que não estão no banco (spawnados por admin, NPC, etc.) são modificados mas **não** têm os mods salvos.

---

## Entrypoints para outros recursos

### Callback `mri_Qbox:customs:client` (cliente)

Abre o menu de customs no cliente sem passar pela zona. Usado por integrações da suite MRI (por exemplo, um ponto de target próprio).

```lua
lib.callback.await('mri_Qbox:customs:client', playerId)
```

### Callback `qbx_customs:client:zone` (cliente)

Retorna o índice da zona em que o jogador está, ou `nil`.

```lua
local zoneId = lib.callback.await('qbx_customs:client:zone', playerId)
```

### Callback `qbx_customs:client:vehicleProps` (cliente)

Retorna `lib.getVehicleProperties` do veículo atualmente no menu.

```lua
local props = lib.callback.await('qbx_customs:client:vehicleProps', playerId)
```

### Callback `qbx_customs:server:pay` (servidor)

Cobra por um mod. `mod` é `'cosmetic'`, `'colors'` ou o ID numérico (`11`, `12`, `13`, `15`, `18`); `level` é o nível instalado. Retorna `true` se pago (ou gratuito pelo job).

```lua
local paid = lib.callback.await('qbx_customs:server:pay', false, mod, level)
```

### Callback `qbx_customs:server:repair` (servidor)

Cobra o reparo com base no `bodyHealth` atual. Retorna `true` se pago ou gratuito.

```lua
local paid = lib.callback.await('qbx_customs:server:repair', false, GetVehicleBodyHealth(vehicle))
```

### Evento `qbx_customs:server:saveVehicleProps`

Salva os mods do veículo em `player_vehicles`, se a placa existir.

```lua
TriggerServerEvent('qbx_customs:server:saveVehicleProps')
```

---

## Localização

Strings via `ox_lib` locale, em `locales/`:

`de`, `en`, `fr`, `pt-br`

Idioma ativo definido no `server.cfg`:

```
setr ox:locale "pt-br"
```

---

## Estrutura de arquivos

```
qbx_customs/
├── client/
│   ├── menus/
│   │   ├── main.lua           — menu raiz, reparo, controle de câmera, save ao fechar
│   │   ├── performance.lua    — motor, freios, transmissão, suspensão, turbo
│   │   ├── parts.lua          — mods cosméticos
│   │   ├── colors.lua         — pintura, película, placa, fumaça de pneu
│   │   ├── paint.lua          — seleção de tinta por acabamento
│   │   ├── neon.lua           — posições e cores de neon
│   │   ├── wheels.lua         — tipos e modelos de roda
│   │   └── extras.lua         — extras do veículo
│   ├── enums/
│   │   ├── VehicleClass.lua   — enum de classes de veículo
│   │   └── WheelType.lua      — enum de tipos de roda
│   ├── dragcam.lua            — câmera orbital com mouse e zoom
│   ├── utils.lua              — GetModLabel, InstallMod (chama o pagamento)
│   └── zones.lua              — polyzones, blips, TextUI, callbacks de zona
├── server/
│   └── main.lua               — cobrança, reparo, save dos mods em player_vehicles
├── config/
│   ├── client.lua             — moeda, catálogo de mods, pinturas, neon, rodas
│   └── shared.lua             — debug, zonas e preços
├── locales/
│   └── *.json                 — traduções (4 idiomas)
├── stream/
│   └── vehicle_paint_ramps.ytd — texturas das pinturas Chameleon
├── carcols_gen9.meta          — definições de cores gen9
├── carmodcols_gen9.meta       — definições de cores dos mods gen9
├── types.lua                  — anotações LuaLS de ZoneOptions e blipOptions
└── fxmanifest.lua
```
