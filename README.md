# docker compose による　ethereum validatorの構築
- docker compose (on linux) コマンドが使える環境が必要です、事前にご準備ください
[docker公式サイト](https://docs.docker.com/engine/install/)

## 材料
1. レポジトリをクローンし、そのディレクトリに移動します
    ```sh
    git clone https://github.com/peko-hr/ethereum-validator.git
    cd ethereum-validator
    ```

## 準備
### create keys
validator_keysディレクトリ内にdeposit.jsonとkeysrore.jsonを作ることが目的です
そのため、セキュリティを考えると別のマシンで実施すべきで、その場合上記２つのファイルを持ってくることでこれはスキップできます（すべきです）

    ```sh
    docker run -it --rm -v $(pwd)/validator_keys:/app/validator_keys ethereum/staking-deposit-cli new-mnemonic \
    --num_validators 1 \
    --chain goerli \
    --eth1_withdrawal_address {{あなたのウォレットアドレス}}
    ```

### import keys
1. validator_keysディレクトリ内のkeysrore.jsonから鍵をインポートします、これによりwalletディレクトリ内に情報が書き込まれます
    ```
    docker run --rm -it \
        -v "$(pwd)/validator_keys:/keys" \
        -v "$(pwd)/wallet:/wallet" \
        -v "$(pwd)/wallet-password.txt:/wallet-password.txt:ro" \
        gcr.io/prysmaticlabs/prysm/validator:stable \
        accounts import \
        --accept-terms-of-use \
        --keys-dir=/keys \
        --wallet-dir=/wallet \
        --wallet-password-file=/wallet-password.txt
    ```

1. 設定したウォレットパスワードを書込みます、以下のファイルにパスワードだけを書き込みます  
    `wallet-password.txt`

    これをしない場合、起動のたびにパスワードを入力する必要があります

### import成功の確認
1. 正しくインポートされたかどうかをリスト形式で確認します
    ```
    docker run --rm -it \
        -v "$(pwd)/wallet:/wallet" \
        -v "$(pwd)/wallet-password.txt:/wallet-password.txt:ro" \
        gcr.io/prysmaticlabs/prysm/validator:stable \
        accounts list \
        --accept-terms-of-use \
        --wallet-dir=/wallet \
        --wallet-password-file=/wallet-password.txt
    ```

## validatorの起動
- beacon-nodeのIPを`mainnet.env`または`holeskyli.env`の`BEACON_RPC`に設定します
- 必要に応じて`conf.d`配下の`validator.yaml`と`recipient.yaml`を編集します
    - 例えば、EL報酬振込先（fee-recipient）とかです、私にくれるならそのままでOKです
- さぁ起動です、もちろんdepositとかは終わってますよね？

    メインネットの場合
    ```sh
    docker compose --env-file mainnet.env up -d
    ```
    ※goerliの場合
    ```sh
    docker compose --env-file holesky.env up -d
    ```






