# AWS Amplify と Amazon Chime SDK を使ってサーバーレスビデオミーティングアプリを作ってみた

こんにちは！ [**株式会社スタートアップテクノロジー**](https://startup-technology.com/) テックリード、 [**AWS Serverless HERO**](https://aws.amazon.com/developer/community/heroes/hidetoshi-matsui/) の松井です！

私は以前から AWS の各種マネージドサービスを自分のアプリケーションに組み込んで遊んできましたが、特にその中でも特化した専門知識がなくても、基本的な開発スキルのみで簡単に自分のアプリケーションに動画配信や双方向の通信機能を追加できるメディア系のサービスにロマンを感じており、以前から [**Amazon Interactive Video Service (IVS) と AWS Amplify を使って自分だけのオリジナル配信サイトを作る！**](https://aws.amazon.com/jp/builders-flash/202107/amplify-ivs-streaming-website) や [**オリジナル配信サイトに視聴者数を取得して表示する機能をコマンド一発で構築する**](https://aws.amazon.com/jp/builders-flash/202202/ivs-display-viewer-command) などの記事を公開させていただいたり、 [**JAWS-UG**](https://jaws-ug.jp) のオンラインイベント、 [**JAWS DAYS 2021 re:Connect**](https://jawsdays2021.jaws-ug.jp/) や [**JAWS PANKRATION 2021 Up till Down**](https://jawspankration2021.jaws-ug.jp/) で配信システムの構築を担当したりしてきました。

