BATAILLION Alice  
MOREAU Marianne  
4 ETI 

# TP 5 - Services réseau

## Exercice 1. Adressage IP (rappels)  
**Vous administrez le réseau interne 172.16.0.0/23 d’une entreprise, et devez gérer un parc de 254 machines
réparties en 7 sous-réseaux. La répar tition des machines est la suivante :**  
**- Sous-réseau 1 : 38 machines**  
**- Sous-réseau 2 : 33 machines**  
**- Sous-réseau 3 : 52 machines**  
**- Sous-réseau 4 : 35 machines**  
**- Sous-réseau 5 : 34 machines**  
**- Sous-réseau 6 : 37 machines**  
**- Sous-réseau 7 : 25 machines**  
**Donnez, pour chaque sous-réseau, l’adresse de sous-réseau, l’adresse de broadcast (multidiffusion) ainsi
que les adresses de la première et dernière machine configurées (précisez si vous utilisez du VLSM ou pas).**  

- Sous-réseau 1 : 172.16.1.0 /26  
- Sous-réseau 2 : 172.16.1.64 /26   
- Sous-réseau 3 : 172.16.1.192 /26  
- Sous-réseau 4 : 172.16.1.128 /26  
- Sous-réseau 5 : 172.16.0.64 /26  
- Sous-réseau 6 : 172.16.0.96 /26  
- Sous-réseau 7 : 172.16.0.16 /27

| Sous-réseau  | Adresse de sous-réseau | Adresse de broadcast | Première adresse | Dernière Adresse |
| ------------ |------------------------| -------------------- | -----------------|------------------|
| 1 | 172.16.1.0 /26 |  Aligné à droite |Aligné à droite |Aligné à droite |
| 2 | 172.16.1.64 /26 |   Aligné à droite |Aligné à droite |Aligné à droite |
| 3 | 172.16.1.192 /26 |    Aligné à droite |Aligné à droite |Aligné à droite |
| 4 | 172.16.1.192 /26 |  Aligné à droite |Aligné à droite |Aligné à droite |
| 5 | 172.16.0.64 /26 |   Aligné à droite |Aligné à droite |Aligné à droite |
| 6 | 172.16.0.96 /26 |    Aligné à droite |Aligné à droite |Aligné à droite |
| 7 | 172.16.0.16 /27 |    Aligné à droite |Aligné à droite |Aligné à droite |
