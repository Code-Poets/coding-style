<!-- $size: 16:9 -->

# Tech talk
## Adresy i transakcje w Bitcoinie i Ethereum

---
# Plan prezentacji
### 1. Adresy kont i transakcje w Ethereum
### 2. Bitcoin: adresy? Jakie adresy?
### 3. Jak sprawdzić stan konta w praktyce

---
# Adresy kont i transakcje w Ethereum

---
## Struktura transakcji Ethereum

| pole         | opis
|--------------|-----------------------------------------------------
| `nonce`      | Numer kolejny transakcji
| `gasprice`   | Cena płacona za krok obliczeń (gaz)
| `startgas`   | Maksymalna ilość kroków obliczeń
| `to`         | Adres docelowy
| `value`      | Przelewana wartość
| `data`       | Dane wywołania transakcji lub kod kontraktu
| `v`          | ECDSA public key recovery byte
| `r`          | Składowa podpisu ECDSA
| `s`          | Składowa podpisu ECDSA

---
## Struktura transakcji Ethereum: Przykład

```
nonce:    0,
gasprice: 50000000000000,
startgas: 21000,
to:       "0x5df9b87991262f6ba471f09758cde1c0fc1de734",
value:    31337
data:     "0x",
v:        "0x1c",
r:        "0x88ff6cf0fefd94db46111149ae4bfc179e9b94721fffd821d38d16464b3f71d0",
s:        "0x45e0aff800961cfce805daef7016b9b675c137a6a41a548f7b60a3484c06a33a",
```

- Adres docelowy: pole `to`
- Adres źródłowy: możliwy do wyliczenia z podpisu (`v`, `r`, `s`).
- Wartość wyrażona w Wei (1 ETH = $10^{-18}$ Wei)

---
## Krzywa eliptyczna

Krzywa postaci $y^2 = x^3 + ax + b$

<center>

![](https://cdn.arstechnica.net/wp-content/uploads/2013/10/elliptic-curve-crypt-image00.png)

</center>

---
## Elliptic Curve Discrete Logarithm Problem
- Krzywa w ciele skończonym
    - Współrzędne są liczbami całkowitymi modulo $p$
- Mnożenie przez skalar zawsze daje punkt, który również jest na krzywej
    $$ Q = m \times P $$
- Odzyskanie $m$ dla znanych $Q$ i $P$ jest trudne obliczeniowo

---
## ECDSA
- Elliptic Curve Digital Signature Algorithm
- $G$ - z góry znany punkt na krzywej, stała algorytmu
- Klucz prywatny: liczba całkowita
    $$ privatekey = d_A $$
- Klucz publiczny: punkt na krzywej
    $$ publickey = d_A \times G $$

---
## Podpis ECDSA
- `r` - współrzędna $x$ losowego punktu $k \times G$ na krzywej
- `s` - obliczone z klucza prywatnego $d_A$, hasha podpisywanych danych oraz `r`
- `v` - wartość pozwalająca odzyskać $k \times G$ z samego `r`
    - Na krzywej eliptycznej więcej niż jedna wartość $y$ może pasować do danego $x$ 

---
## Krzywa `secp256k1`

$$ y^2 = x^3 + 7 $$

- Znana też jako Koblitz Curve lub Bitcoin Curve 
- Zdefiniowana w ciele skończonym o $p$ rzędu $2^{256}$
<center>

![](https://steemitimages.com/DQmbHgDo7kfoTbn1WtgJTwaQ9mKpUBCbcor1apbbD2GVuZt/image.png)

</center>

---
## Adresy Ethereum

- Właściciel adresu dysponuje parą kluczy ECDSA
- Adres jest hashem klucza publicznego
    $$ address = \text{hash}(publickey)$$
- (`v`, `r`, `s`) uzyskujemy podpisując (`nonce`, `gasprice`, `startgas`, `to`, `value`, `data`) kluczem prywatnym, dowodząc że go posiadamy
- Stan konta = różnica sumarycznej wartości transakcji do i z adresu

---
#  Bitcoin: adresy? Jakie adresy?

---
## Struktura transakcji Bitcoin (pre-SegWit)

| pole         | opis
|--------------|-----------------------------------------------------
| wersja       | Wersja, obecnie zawsze `1`
| in-counter   | Liczba wejść
| inputs       | Lista wejść transakcji
| out-counter  | Liczba wyjść
| outputs      | Lista wyjść transakcji
| lock_time    | Numer bloku/timestamp kiedy najwcześniej transakcja może zostać wykopana

---
## Struktura transakcji Bitcoin: Przykład

```
Input:
Previous tx: f5d8ee39a430901c91a5917b9f2dc19d6d1a0e9cea205b009ca73dd04470b9a6
Index: 0
scriptSig: 304502206e21798a42fae0e854281abd38bacd1aeed3ee3738d9e1446618c4571d10
90db022100e2ac980643b0b82c0e88ffdfec6b64e3e6ba35e7ba5fdd7d5d6cc8d25c6b241501

Output:
Value: 5000000000
scriptPubKey: OP_DUP OP_HASH160 404371705fa9bd789a2fcd52d2c580b65d35549d
OP_EQUALVERIFY OP_CHECKSIG
```
- Dla uproszczenia tylko jedno wejście i wyjście
- Wejście wskazuje na wyjście innej transakcji, nie adres
- Brak adresu docelowego. Wyjście zdefiniowane jest skryptem
- Wartość wyrażona w Satoshi (1 BTC = $10^{-8}$ Satoshi)

---
## Bitcoin script

- Język programowania oparty na stosie (jak Forth lub PostScript)
- Nie jest kompletny w sensie Turinga - brak pętli
- Nie wszystkie operacje zdefiniowane w protokole są obecnie dozwolone


---
## Bitcoin script - operacje
- Stałe
- Warunki
- Manipulacja stosem (przesuń, powiel, zamień, itp.)
- Operacje arytmetyczne i logiczne (mnożenie i dzielenie zabronione)
- Operacje bitowe (obecnie w większości zabronione)
- Operacje na ciągach znaków (obecnie w większości zabronione)
- Operacje kryptograficzne (hashowanie, podpisy)

---
## Wykonanie programu w języku opartym na stosie

```
| stos    | skrypt                  |
|---------|-------------------------|
| []      | [1 2 DUP ADD ADD 5 CMP] |
| [1]     |   [2 DUP ADD ADD 5 CMP] |
| [1 2]   |     [DUP ADD ADD 5 CMP] |
| [1 2 2] |         [ADD ADD 5 CMP] |
| [1 4]   |             [ADD 5 CMP] |
| [5]     |                 [5 CMP] |
| [5 5]   |                   [CMP] |
| true    |                      [] |
```

---
## Parametry programu języku opartym na stosie

- Aby przekazać parametry wystarczy położyć je na stosie przed programem
- Wykonanie skryptu `DUP ADD` z parametrem `1`:
```
| stos    | skrypt      |
|---------|-------------|
| []      | [1 DUP ADD] |
| [1]     |   [DUP ADD] |
| [1 1]   |       [ADD] |
| [2]     |          [] |
```

---
## Typowa transakcja Bitcoin: wejścia
```
Previous tx: f5d8ee39a430901c91a5917b9f2dc19d6d1a0e9cea205b009ca73dd04470b9a6
Index: 0
scriptSig: 304502206e21798a42fae0e854281abd38bacd1aeed3ee3738d9e1446618c4571d10
90db022100e2ac980643b0b82c0e88ffdfec6b64e3e6ba35e7ba5fdd7d5d6cc8d25c6b241501
```
- `Previous tx` i `Index` wskazują na wyjście jednej z już wykopanych transakcji
- `scriptSig` określa parametry wejściowe dla skryptu zdefiniowanego dla tego wyjścia
    - Podpis
    - Klucz publiczny

---
## Typowa transakcja Bitcoin: wyjścia

```
Value: 5000000000
scriptPubKey: OP_DUP OP_HASH160 404371705fa9bd789a2fcd52d2c580b65d35549d
OP_EQUALVERIFY OP_CHECKSIG
```
- `scriptPubKey` określa skrypt dla danego wyjścia
- Wyjścia może użyć w swojej transakcji każdy, kto poda takie parametry, że skrypt zwróci `true`
- Skrypt w tej transakcji sprawdza podpis
    - Hash klucza publicznego jest zahardkodowany w skrypcie

---
## Adresy Bitcoinowe
- Adresy nie istnieją
- Dając komuś nasz "adres", dajemy mu tak naprawdę hash klucza publicznego
- Skrypt może być dowolnie skonstruowany i brać dowolne parametry. Nie musi zawierać hasha klucza publicznego albo może zawierać ich wiele
    - Problem z określeniem odbiorcy w ogólnym przypadku
- Przykłady skryptów:
    - Multisig (kontrakt)
    - Transakcja, której wyjścia może wydać każdy
    - Transakcja, której wyjścia nie może wydać nikt
    - Zadanie matematyczne
    
---
## Wielokrotne wykorzystanie tego samego "adresu"

### Jak to działa?
- Nie da się powielić wyjścia transakcji
    - Nowa transakcja = nowe wyjścia
- Większość klientów udaje, że adresy istnieją
    - Wydanie środków z adresu powoduje użycie w transakcji wszystkich znanych wyjść, których skrypt zawiera hash tego samego klucza prywatnego

---
## Wielokrotne wykorzystanie tego samego "adresu": zagrożenia

- Osłabienie klucza
    - Algorytm podpisu w Bitcoinie to też ECDSA
    - Podpis nie jest deterministyczny - zależy od losowo generowanego punktu $k \times P$
    - Dostęp do wielu różnych podpisów wykonanych tym samym kluczem ułatwia jego złamanie
- Mniejsza prywatność: łatwiej wydedukować, które adresy mają tego samego właściciela

---
# Jak sprawdzić stan konta w praktyce

---
## W UI klienta

- UI może łączyć się z lokalnym lub zdalnym węzłem
- Węzeł może być pełnym lub lekkim klientem

### Bitcoin
- Bitcoin Wallet
- Electrum
- Mycelium

### Ethereum
- Mist
- MyEtherWallet

---
## Online
### Bitcoin blockchain browsers
- Lista: [Bitcoin blockchain browsers](https://en.bitcoin.it/wiki/Category:Block_chain_browsers)
- Transakcja z przykładu: https://www.blockchain.com/pl/btc/tx/5a4ebf66822b0b2d56bd9dc64ece0bc38ee7844a23ff1d7320a88c5fdb2ad3e2


### Ethereum blockchain browsers
- Popularne strony:
    - https://etherchain.org
    - https://etherscan.io
- Transakcja z przykładu: https://etherscan.io/tx/0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060

---
# Źrodła
---
## Transakcje Ethereum
- [Ethereum transaction structure](https://lsongnotes.wordpress.com/2017/12/21/ethereum-transaction-structure/)

## Transakcje Bitcoinowe
- [Bitcoin Wiki > Transaction](https://en.bitcoin.it/wiki/Transaction)
- [Bitcoin Wiki > Script](https://en.bitcoin.it/wiki/Script)
- [Bitcoin Wiki > Address reuse](https://en.bitcoin.it/wiki/Address_reuse)

---
## Krzywe eliptyczne
- [ECDSA: (v, r, s), what is v?](https://bitcoin.stackexchange.com/questions/38351/ecdsa-v-r-s-what-is-v)
- [Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
- [A (relatively easy to understand) primer on elliptic curve cryptography](https://arstechnica.com/information-technology/2013/10/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/)
- [Calculate BITCOIN PublicKey](https://steemit.com/ellipticcurve/@sso/calculate-bitcoin-publickey) - jak ręcznie policzyć wyliczyć klucz z użyciem krzywej eliptycznej `secp256k1`

---
## Klienci
- [Bitcoin Wiki > Thin client security](https://en.bitcoin.it/wiki/Thin_Client_Security)
- [A Primer on Ethereum Blockchain Light Clients](https://medium.com/zkcapital/a-primer-on-ethereum-blockchain-light-clients-f3cadde49137)
- [Bitcoin Wiki > Clients](https://en.bitcoin.it/wiki/Clients)
- [Ethereum Wiki > Clients, tools, dapp browsers, wallets and other projects](https://github.com/ethereum/wiki/wiki/Clients,-tools,-dapp-browsers,-wallets-and-other-projects)
