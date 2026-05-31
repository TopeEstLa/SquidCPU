# SquidCPU

Projet Logisim d'un processeur simplifié **CYT-VX8** découpé en blocs indépendants : ALU, banc de registres, chemin de données complet (ALU + registres), décodeur d'instructions et registre de fetch/decode.

Ce processeur est basé sur l'architecture Harvard (mémoire de code et mémoire de données séparées). Toutes les instructions sont encodées sur **24 bits** fixes. Tous les registres de données sont sur **8 bits**, sauf le registre de comparaison RCMP qui n'utilise qu'**1 bit**.

---

## Fichiers du projet

| Fichier | Rôle |
|---|---|
| `ALU.circ` | Bloc ALU seul (opérations arithmétiques et logiques) |
| `cpu_registry.circ` | Registres réutilisables du processeur (1 bit, 8 bits, banc, 24 bits) |
| `alu_registry.circ` | Assemblage ALU + banc de registres (chemin de données complet) |
| `decode.circ` | Décodeur complet : registre fetch/decode + logique de décodage |
| `decodecollapse.txt` | Table de vérité Logisim du bloc `type_op_decode` |

---

## ALU (`ALU.circ`)

L'ALU possède deux entrées de données 8 bits (`A` et `B`) et une entrée de sélection `slct` sur 4 bits.

### Décodage de `slct`

Les 2 bits de gauche choisissent le bloc fonctionnel, et les 2 bits de droite choisissent l'opération dans ce bloc.

- `00` : bloc arithmétique
  - `00` : `ADD`
  - `01` : `SUB`
- `01` : bloc logique
  - `00` : `AND`
  - `01` : `OR`
  - `10` : `XOR`
- `10` : bloc comparaison
  - `00` : `CMP_EQ`
  - `01` : `CMP_NE`
  - `10` : `CMP_LT`
  - `11` : `CMP_GT`
- `11` : bloc shift/rotate
  - `00` : `LSL`
  - `01` : `LSR`
  - `10` : `ROR`
  - `11` : `ROL`

### Sorties

- `S` : résultat 8 bits.
- `Err` : bit d'erreur.

---

## Registres CPU (`cpu_registry.circ`)

Ce fichier regroupe les registres réutilisables à travers tout le processeur :

- registre 1 bit,
- registre 8 bits,
- banc de registres 7×8 bits + 1×1 bit,
- registre 24 bits.

Ces blocs servent à la fois au stockage de données, au suivi d'état et au registre de fetch/decode.

---

## Assemblage ALU + Registres (`alu_registry.circ`)

Ce circuit regroupe l'ALU et les registres nécessaires à son utilisation.

### Entrées principales

- `Rin1` : sélection du registre source A.
- `Rin2` : sélection du registre source B.
- `Rout` : sélection du registre lu en sortie.
- `REG_in` : sélection de l'entrée chargée dans le registre cible.
- `op` : sélection de l'opération ALU.

### Sorties

- `zero` : drapeau zéro.
- `REG_out` : valeur lue depuis le banc de registres.
- `ALU_out` : résultat de l'ALU.

---

## Décodeur (`decode.circ`)

Le décodeur lit une instruction de 24 bits et produit tous les signaux de contrôle nécessaires à l'exécution. Il est composé de deux sous-blocs.

### Format d'une instruction (24 bits)

```
type_op  |   op  |   Rin1  |   Rin2/Imm  |  0000  |  Rout
4 bits    4 bits    4 bits     4 bits      4 bit     4 bits
```

Pour les instructions spéciales (`JUMP`, `LOAD`, `STOR`, `CNST`), les bits 15 à 4 contiennent une adresse (`ADDR8`) ou une constante (`CNST8`) à la place des champs registres.

### Sous-bloc `fetch_decode`

Simple registre 24 bits (registre `IF`) qui mémorise l'instruction lue depuis la RAM de code, afin qu'elle soit disponible pour le décodage au cycle suivant.

### Sous-bloc `type_op_decode`

Bloc purement combinatoire. Il prend en entrée les 8 premiers bits de l'instruction (`type_op[3..0]` et `op[3..0]`) et produit les signaux de contrôle de base.

La table de vérité complète est dans `decodecollapse.txt`.

**Sorties de `type_op_decode` :**

| Signal | Taille | Rôle |
|---|---|---|
| `slcOp[3..0]` | 4 bits | Sélection de l'opération ALU |
| `slctREG_in[1..0]` | 2 bits | Source à charger dans le registre destination (`00`=ALU, `01`=RAM_OUT, `10`=CNST8) |
| `jump` | 1 bit | Demande de saut |
| `stor` | 1 bit | Demande d'écriture en RAM de données |

### Sous-bloc `decode_instru`

Combine `type_op_decode`, des multiplexeurs et les champs de l'instruction pour produire les signaux finaux envoyés au reste du processeur.

**Sorties de `decode_instru` :**

| Signal | Rôle |
|---|---|
| `slctOp` | Opération ALU |
| `slctREG_in` | Source d'entrée du registre destination (ALU / LOAD / CNST) |
| `jump` | Activation du saut |
| `slctRegA` | Numéro du registre source A |
| `stor` | Activation de l'écriture en RAM de données |
| `ADDR8` | Adresse ou constante 8 bits extraite de l'instruction |
| `slctRegB` | Numéro du registre source B |
| `slctRegWrite` | Numéro du registre destination à écrire |

---

## Table de vérité (`decodecollapse.txt`)

Fichier exporté depuis Logisim, à garder synchronisé avec le circuit `type_op_decode` dans `decode.circ`. Il peut être réimporté directement dans Logisim pour regénérer ou vérifier le bloc.

---

## OpCodes supportés

Le décodeur couvre les familles d'opcodes suivantes :

`ADD`, `SUB`, `AND`, `OR`, `XOR`, `CMP_EQ`, `CMP_NE`, `CMP_GT`, `CMP_LT`, `LSL`, `LSR`, `ROR`, `ROL`, `JUMP`, `LOAD/CNST`, `STOR`, `CNST8/ADDR8`

### `type_op = 0001` — Opérations ALU à 2 registres

`ADD`, `SUB`, `AND`, `OR`, `XOR` : le champ `op` sélectionne l'opération, `slcOp` est transmis directement à l'ALU, le résultat est écrit dans `Rout` (`slctREG_in = 00`).

### `type_op = 0000` — Comparaisons et shifts

`CMP_EQ`, `CMP_NE`, `CMP_GT`, `CMP_LT` : le registre destination est toujours `RCMP` (id 7), câblé en dur dans l'opcode (`Rout = 0111`).

`LSL`, `LSR`, `ROR`, `ROL` : opérations sur un seul registre source, `Rin2` est mis à `0000`.

### `type_op = 0010` — Instructions mémoire et saut

- `LOAD` : lit un octet depuis la RAM de données à l'adresse `ADDR8` et l'écrit dans `Rout`. `slctREG_in = 01`.
- `CNST` : charge la constante 8 bits `CNST8` extraite de l'instruction dans `Rout`. `slctREG_in = 10`.
- `STOR` : écrit la valeur de `Rin1` dans la RAM de données à l'adresse `ADDR8`. Active `stor = 1`.
- `JUMP` : active `jump = 1`, l'Unité d'Adressage utilise `ADDR8` comme nouvelle valeur du PC.

---

## Fichiers de travail

- `decodecollapse.txt` : à maintenir synchronisé avec `type_op_decode` dans `decode.circ`.
- Les fichiers `.circ` sont des circuits Logisim à ouvrir avec Logisim Evolution.