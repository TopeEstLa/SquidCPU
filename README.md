# SquidCPU

## ALU

Cette ALU possède deux entrées 8 bits, `A` et `B`, et une entrée de sélection `slct` sur 4 bits.

### Décodage de `slct`

`slct` est sur 4 bits : les 2 bits de gauche choisissent le bloc, et les 2 bits de droite choisissent l'opération dans ce bloc.

- `00` : `alu_arithm`
	- `00` : `ADD`
	- `01` : `SUB`
- `01` : `alu_logic`
	- `00` : `AND`
	- `01` : `OR`
	- `10` : `XOR`
- `10` : `alu_comp`
	- `00` : `EQUAL`
	- `01` : `DIFF`
	- `10` : `INF`
	- `11` : `SUP`
- `11` : `bloc_shift`
	- `00` : `LSL`
	- `01` : `LSR`
	- `10` : `ROR`
	- `11` : `ROL`

Certaines combinaisons peuvent rester inutilisées selon le bloc.

### Sorties

- `S` : résultat 8 bits
- `Err` : bit d'erreur

### Bonus

- Sortie erreur
- `LSL`, `LSR`, `ROL`, `ROR`

### Fichiers Circuit

- `registry_file.circ` : les registres
- `alu_registry.circ` : assemblage ALU et registre
- `ALU.circ` et `registry_file.circ` sont utilisés en library dans le fichier d'assemblage
