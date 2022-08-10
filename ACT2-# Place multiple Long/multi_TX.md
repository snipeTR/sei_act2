## Place multiple Long/Short orders in one transaction (bundled order placement) in any market on Vortex. Currently this needs to be done via CLI
cüzdanınız yüklü değilse yükleyin.
```bash
    seid keys add wallet --recover
```

## dosyaları silin
daha önce bu dosyalardan oluşturduysanız silin.
```bash
cd $HOME
rm txs.json
rm gen_tx.json
```

## HAZIRLIKLAR
aşağıdaki komutları tek tek sunucuya yapıştırın.
bu satırlar ile değişkenlerinizi ayarlıyorsunuz.
aşağıdaki komutlarda "> ve <" karakterlerini **koymayacaksınız**.
#### değişken ayarlama
aşağıdakilerin her satırı tek komuttur. satır satır kopyalayın.
```bash
memo=> **buraya memo yazacaksınız. genelde discord adresiniz.** <
ADDR=> **sei adresiniz** <
ACC=$(seid q account $ADDR -o json | jq -r .account_number)
seq=$(seid q account $ADDR -o json | jq -r .sequence)
wallet=> **cüzdan adınızı yazacaksınız örneğin : *wallet* ** <
```
#### gen_tx.json dosyası oluşturma
aşağıdaki kısmı ilk karakterden son karaktere kadar tek seferde sunucuya yapıştırın.
```bash
echo '{
  "body": {
    "messages": [
{
  "@type": "/seiprotocol.seichain.dex.MsgPlaceOrders",
  "creator": "$ADDR",
  "orders": [
    {
      "id": "0",
      "status": "PLACED",
      "account": "$ADDR",
      "contractAddr": "sei14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9sh9m79m",
      "price": "11.800000000000000000",
      "quantity": "0.005000000000000000",
      "priceDenom": "USDC",
      "assetDenom": "ATOM",
      "orderType": "MARKET",
      "positionDirection": "LONG",
      "data": "{\"position_effect\":\"Open\",\"leverage\":\"1\"}",
      "statusDescription": ""
    }
  ],
  "contractAddr": "sei14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9sh9m79m",
  "funds": [
    {
      "denom": "uusdc",
      "amount": "56000"
    }
  ],
  "autoCalculateDeposit": false
},
{
  "@type": "/seiprotocol.seichain.dex.MsgPlaceOrders",
  "creator": "$ADDR",
  "orders": [
    {
      "id": "0",
      "status": "PLACED",
      "account": "$ADDR",
      "contractAddr": "sei14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9sh9m79m",
      "price": "11.000000000000000000",
      "quantity": "0.005000000000000000",
      "priceDenom": "USDC",
      "assetDenom": "ATOM",
      "orderType": "MARKET",
      "positionDirection": "SHORT",
      "data": "{\"position_effect\":\"Open\",\"leverage\":\"1\"}",
      "statusDescription": ""
    }
  ],
  "contractAddr": "sei14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9sh9m79m",
  "funds": [
    {
      "denom": "uusdc",
      "amount": "56000"
    }
  ],
  "autoCalculateDeposit": false
}
    ],
    "memo": "$memo",
    "timeout_height": "0",
    "extension_options": [],
    "non_critical_extension_options": []
  },
  "auth_info": {
    "signer_infos": [],
    "fee": {
      "amount": [
        {
          "denom": "usei",
          "amount": "0"
        }
      ],
      "gas_limit": "0",
      "payer": "",
      "granter": ""
    }
  },
  "signatures": []
}' > $HOME/gen_tx.json
```
## KONTROL
yukarıdaki komut ile sunucunuzda gen_tx.json isimli bir dosya oluşturmuş olmalısınız.
aşağıdaki komutların herhangi birisi ile dosyanın doğru oluşup oluşmadığını kontrol edin. [örnek dosya](https://github.com/snipeTR/sei_act2/blob/main/ornek_gen_tx.json "örnek dosya")
creator ve account kısmında kendi sei adresiniz yazmalı. memo kısmında ise kendi yazdığınız memo verisinin yazıyor olması lazım. kontrollerinizi dikkatlice yapın. gerekliyse düzeltmeleri nano aracılığı ile yapın ve dosyayı kaydedin. herşey doğru gözüküyorsa bir değişiklik yapmanıza gerek yok.

```bash
cat gen_tx.json | jq .
```
yada
```bash
nano gen_tx.json
```

## İMZALAMA
aşağıdaki komut ile birlikte yukarıda oluşturduğunuz gen_tx.json dosyasını imzalayacaksınız. bu imzalama işlemi için daha önce girdiğiniz sei adresine ait olan cüzdanın seid programına yüklü olması gerekiyor.

```bash
seid tx sign $HOME/gen_tx.json -s $seq -a $ACC --offline --from $wallet --chain-id atlantic-1 --output-document $HOME/txs.json
```

## YAYINLAMA
aşağıdaki komut ile birlikte  daha önce imzaladığınız ve txs.json dosyasına imzaladığınız TX bilgilerinizi sisteme yayınlıyorsunuz.

```bash
seid tx broadcast $HOME/txs.json
```
eğer aşağıdaki gibi bir çıktı alıyorsanız ve code değeri 0 (sıfır) olarak gözüküyorsa işleminiz başarı ile tamamlanmıştır. (**AŞAĞIDAKİLER KOMUT DEĞİLDİR.**)

```bash
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: 5BA-SAMPLE-SAMPLE-SAMPLE-SAMPLE-SAMPLE-SAMPLE-SAMPLE-SAMPLE-C29C
```

## HATA DURUMUNDA KONTROL EDİLECEKLER.
### CODE:32
sequance hatasıdır. değişken ayarlama adımda, yanlış bir sıra değeri belirttiniz. Tx şablonunu doğru sıra değeriyle yeniden imzalayın. 
aşağıdaki komuttan itibaren işlemleri tekrarlayın.
```bash
seq=$(seid q account $ADDR -o json | jq -r .sequence)
```
### CODE:19
bu imza zaten mempool sırasında olan bir işlem. aşağıdaki komutları tekrar çalıştırın.
```bash
seq=$(seid q account $ADDR -o json | jq -r .sequence)
```

```bash
seid tx sign $HOME/gen_tx.json -s $seq -a $ACC --offline \
--from $wallet --chain-id atlantic-1 \
--output-document $HOME/txs.json
```

```bash
seid tx broadcast $HOME/txs.json
```
### CODE:4
imzalama hatası. code değeri 4 veriyorsa ya sei adresinizi yanlış yazdınız. yada kendinize ait olmalayan yada sunucuda kurulu olmayan bir adres ile imzalamaya çalışıyorsunuz. lütfen wallet adını, sei adresinizi ve gen_tx.json dosyasına yazılı sei adreslerini kontrol ediniz.
```bash
seid keys list
```
### CODE:13
hesabınızda yeterli fee ödemek için bakiye yetersiz. lütfen bir miktar fee ödemek için sei jetonunuzun olduğunu kontrol edin. eğer jetonunuz varsa gen_tx.json dosyasındaki kısımdaki amount değerini yükseltin örneğin 200
```bash
 ...........
 "fee": {
      "amount": [
        {
          "denom": "usei",
          "amount": "0"
        }
 ...........
```



Thanks to [craving-for-knowledge](https://pandao.github.io/editor.md/en.html](https://craving-for-knowledge.gitbook.io/craving_for_knowledge/proekty/sei/act-2-missions/place-multiple-orders-in-one-transaction) "craving-for-knowledge") for supporting the creation of this article.
