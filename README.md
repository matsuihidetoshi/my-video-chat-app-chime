# AWS Amplify と Amazon Chime SDK を使ってサーバーレスビデオミーティングアプリを作ってみた

こんにちは！ [**株式会社スタートアップテクノロジー**](https://startup-technology.com/) テックリード、 [**AWS Serverless HERO**](https://aws.amazon.com/developer/community/heroes/hidetoshi-matsui/) の松井です！

私は以前から AWS の各種マネージドサービスを自分のアプリケーションに組み込んで遊んできましたが、特にその中でも、特化した専門知識がなくても、基本的な開発スキルのみで簡単に自分のアプリケーションに動画配信や双方向の通信機能を追加できるメディア系のサービスにロマンを感じており、以前から [**Amazon Interactive Video Service (IVS) と AWS Amplify を使って自分だけのオリジナル配信サイトを作る！**](https://aws.amazon.com/jp/builders-flash/202107/amplify-ivs-streaming-website) や [**オリジナル配信サイトに視聴者数を取得して表示する機能をコマンド一発で構築する**](https://aws.amazon.com/jp/builders-flash/202202/ivs-display-viewer-command) などの記事を公開させていただいたり、 [**JAWS-UG**](https://jaws-ug.jp) のオンラインイベント、 [**JAWS DAYS 2021 re:Connect**](https://jawsdays2021.jaws-ug.jp/) や [**JAWS PANKRATION 2021 Up till Down**](https://jawspankration2021.jaws-ug.jp/) で配信システムの構築を担当したりしてきました。  

そして今回は、[**AWS Amplify**](https://aws.amazon.com/amplify/) や [**Amazon Chime SDK**](https://aws.amazon.com/chime/chime-sdk) を使ったオリジナルのサーバーレスビデオミーティングアプリを構築したので、その概要や工夫したポイントなどについて解説します！  
ソースコードもご紹介しますので、参考にしていただいて実際にご自身でも構築\*していただけるかと思います！

\*ご注意事項
- 実際に今回ご紹介する内容と同等の仕組みを構築してインターネット上に公開する場合、 [**電気通信事業法**](https://www.soumu.go.jp/main_content/000580688.pdf) に基づく届出が必要になる可能性がございますので、ご自身の責任において適切にご判断いただき、運用してください。
- 今回ご紹介する内容を実践していただき、簡易的に動作確認していただく場合、数十円〜数百円程度の AWS 利用料金が発生します。

## 利用するサービスの概要

### Amazon Chime SDK
今回ご紹介するアプリケーションは主に **Amazon Chime SDK** を利用しています。  
**Amazon Chime SDK** は [**Amazon Chime**](https://aws.amazon.com/chime) の様なビデオミーティングの機能をご自身で開発した Web/Mobile アプリケーションに組み込むことができる開発キットです。  
以前ご紹介した **Amazon Interactive Vide Service(IVS)** も、同じ様にご自身の開発したアプリケーションに動画配信機能を組み込むことができますが、実現できる機能に違いがありますので、下記にイメージを記載します。


