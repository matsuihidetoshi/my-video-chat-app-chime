# AWS Amplify と Amazon Chime SDK を使ってサーバーレスビデオミーティングアプリを作ってみた

こんにちは！ [**株式会社スタートアップテクノロジー**](https://startup-technology.com/) テックリード、 [**AWS Serverless HERO**](https://aws.amazon.com/developer/community/heroes/hidetoshi-matsui/) の松井です！

私は以前から AWS の各種マネージドサービスを自分のアプリケーションに組み込んで遊んできましたが、特にその中でも、特化した専門知識がなくても、基本的な開発スキルのみで簡単に自分のアプリケーションに動画配信や双方向の通信機能を追加できるメディア系のサービスにロマンを感じており、以前から [**Amazon Interactive Video Service (IVS) と AWS Amplify を使って自分だけのオリジナル配信サイトを作る！**](https://aws.amazon.com/jp/builders-flash/202107/amplify-ivs-streaming-website) や [**オリジナル配信サイトに視聴者数を取得して表示する機能をコマンド一発で構築する**](https://aws.amazon.com/jp/builders-flash/202202/ivs-display-viewer-command) などの記事を公開させていただいたり、 [**JAWS-UG**](https://jaws-ug.jp) のオンラインイベント、 [**JAWS DAYS 2021 re:Connect**](https://jawsdays2021.jaws-ug.jp/) や [**JAWS PANKRATION 2021 Up till Down**](https://jawspankration2021.jaws-ug.jp/) で配信システムの構築を担当したりしてきました。  

そして今回は、[**AWS Amplify**](https://aws.amazon.com/amplify/) や [**Amazon Chime SDK**](https://aws.amazon.com/chime/chime-sdk) を使ったオリジナルのサーバーレスビデオミーティングアプリを構築したので、その概要や工夫したポイントなどについて解説します！  
**Chime SDK** 単体での利用方法はすでに多くのドキュメントが公開されていますが（実際私も大変参考にさせていただきました）、一つのアプリケーションとしてエンドユーザーに提供することを想定すると、ユーザー登録・ログイン機能やユーザーに紐づくミーティング ID の保存や権限管理なども考える必要が出てくるかと思います。  
今回はそれらも含めたソースコードもご紹介しますので、参考にしていただいて実際にご自身でも構築\*していただけるかと思います！

\*ご注意事項
- 実際に今回ご紹介する内容と同等の仕組みを構築してインターネット上に公開する場合、 [**電気通信事業法**](https://www.soumu.go.jp/main_content/000580688.pdf) に基づく届出が必要になる可能性がございますので、ご自身の責任において適切にご判断いただき、運用してください。
- 今回ご紹介する内容を実践していただき、簡易的に動作確認していただく場合、数十円〜数百円程度の AWS 利用料金が発生します。

## 利用するサービスの概要

### Amazon Chime SDK

今回ご紹介するアプリケーションは主に **Amazon Chime SDK** を使用しています。  
**Amazon Chime SDK** は [**Amazon Chime**](https://aws.amazon.com/chime) の様なビデオミーティングの機能をご自身で開発した Web/Mobile アプリケーションに組み込むことができる開発キットです。  
以前ご紹介した **Amazon Interactive Vide Service(IVS)** も、同じ様にご自身の開発したアプリケーションに動画配信機能を組み込むことができますが、実現できる機能に違いがありますので、下記にイメージを記載します。

![chime-sdk 001](https://user-images.githubusercontent.com/38583473/183244803-7b69c04b-21a8-42fa-97f2-ad967c36b30f.png)

このように、 **IVS** では、単一の配信者が一方通行で複数の視聴者に動画を配信できるのに対して、 **Chime SDK** では双方向の通信=ビデオミーティング機能を実現することができます。  
その際の通信は **WebRTC SFU(Selective Forwarding Unit)** という仕組みになっており、**P2P** の通信と違い、クライアント同士がメッシュ状の経路で通信せず、あくまで単一の **SFU サーバー** のみと各クライアントが通信するため、ミーティング参加者が増えても各々の回線や端末に高い負荷がかかることがない様になっています。

### AWS Amplify

もう一つ重要なツールとして、今回は **AWS Amplify** を使用しています。  
**Amplify** は、Web/Mobile 向けのアプリケーションを迅速に構築するための機能を備えたサービス、 SDK 、 CLI 、などを含む一連のツールチェーンです。  
今回は **Amplify** を使用し、実際にミーティングに参加するための Web フロントエンドのデプロイとホスティング、ユーザーの登録・ログイン、権限に基づいたミーティングに必要な情報の CRUD( データの読み書き ) 機能を構築していきます。
**Amplify** に関して、詳しくは [**こちら**](https://aws.amazon.com/amplify/) をご覧ください。

## 全体構成

今回ご紹介するアプリケーションの全体の構成です。  
**AWS** の各種サービスをビルディングブロックとして組み合わせ、一つのアプリケーションとして機能させていますが、基本的には **Amplify** と　**Serverless Framework** を使って、アプリケーションのソースコードだけでなく、構成自体もコードベースで管理しています。

![E3jjll9VgAMYKz6](https://user-images.githubusercontent.com/38583473/183279387-a47bf6df-81c2-437b-b8eb-114efd58b56f.jpeg)


## 各コンポーネントの解説

全体構成のうち、一つ一つの箇所について解説していきます。

### ミーティングクライアント(Web フロントエンド)
ユーザーがビデオミーティングに参加するためにはクライアントアプリケーションが必要になります。  
今回は **Vue.js** を使って Web アプリケーションとしてフロントエンドを構築し、 **Amplify Console** のホスティング機能を使ってデプロイしています。  
静的コンテンツとしてフロントエンドのアプリケーションをデプロイできれば、例えば **Amazon S3** と **Amazon CloudFront** を組み合わせても良いのですが、 **Amplify Console** を使って **GitHub** と連携させ、アプリケーションのデプロイからホスティング、 CI/CD のパイプラインまでをまとめて構成できるので非常に便利です。
サンプルコードは [**こちら**](https://github.com/matsuihidetoshi/my-video-chat-app-chime-front) です。

### ユーザー登録・ログイン機能
ユーザーがビデオミーティングを作成して主催したり参加したりする際に、ミーティング自体の作成や削除をする必要があります。  
また、例えば作成したミーティングを削除する際に、ミーティングを作ったユーザーのみが削除できる様にしたい場合などに権限管理なども必要になってきます。  
そのために今回は **Amazon Cognito** を使ってユーザー登録・ログイン機能を実現しています。  
こちらは **Amplify CLI** からリソースを作成しており、前述の **ミーティングクライアント(Web フロントエンド)** の [**サンプルコード**](https://github.com/matsuihidetoshi/my-video-chat-app-chime-front) に含まれているため、別途リソースを作成したりする必要はありません。  
また、各種フレームワークに対応した基本的なユーザー登録・ログイン機能については [**こちら**](https://github.com/aws-samples/aws-amplify-auth-starters) にサンプルが紹介されています。

### ミーティング情報の管理機能
ユーザーがミーティングを作成したり、作成済みのミーティングを一覧表示したり、その中からミーティングを選択して参加したりといった機能を実現するために、ミーティングの ID を保存する必要があります。  
今回は **Amazon DynamoDB** をデータソースとし、 **AWS AppSync** を経由してフロントエンドとリアルタイムにミーティングの ID の更新機能を実現しました。  
こちらもユーザー登録・ログイン機能と同様、 **Amplify CLI** を使ってリソースを作成しており、 [**サンプルコード**](https://github.com/matsuihidetoshi/my-video-chat-app-chime-front) にも含まれています。  

### Chime ミーティングハンドラー
**Chime SDK** を使ってミーティングを作成したり、参加者としての ID を発行し、実際にミーティングに参加するための API を **AWS Lambda** と **Amazon API Gateway** を使って構築しました。  
こちらは **Serverless Framework** を使って構築しており、サンプルコードは [**こちら**](https://github.com/matsuihidetoshi/my-video-chat-app-chime-handler) です。
また、前述のミーティングの ID を保持する **DynamoDB** のレコードが削除された際に別の **Lambda** 関数を呼び出し、 **Chime** 上に存在するミーティングも連動して削除する機能も追加しました。  
こちらは、仕様上5分以上ミーティングへの接続がない場合自動的にミーティング自体が削除される仕様になっているので、さほど重要ではないですが、 **DynamoDB Streams** を活用したイベントドリブンなアーキテクチャの検討のために追加してみました。

## まとめ
今回は主に **Chime SDK** と **Amplify** のご紹介でしたが、  
- **Chime SDK** を使えば比較的簡単に独自のアプリケーションにビデオミーティング機能が追加できる。
- **Amplify** を使えば、アプリケーション今回の様に他の AWS サービスと統合して、実際に動くアプリケーションとして、容易に機能の検証ができる。
ということがお分かりいただけたかと思います！    

また、今回のサンプル等は、基本的な Web 開発のスキルと **Amplify** の基本的な理解が必要になるかと思います。  
**Amplify** について詳しく知りたい、そもそもどんなものか分からないという方は、 [**こちら**](https://aws.amazon.com/jp/builders-flash/202103/amplify-app-development) などを参考に別のサンプルなども動かしてみて理解を深めていただけると良いかと思います。  

今回の内容を参考に、ぜひご自身でもお試しいただけると幸いです！

Happy Coding!

