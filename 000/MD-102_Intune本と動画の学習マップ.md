# MD-102 Intune本と動画の学習マップ

対象は『ひと目でわかるIntune 第3版』と、社内ネットナビの公開動画 **Intuneシリーズ第1〜5回、Entra IDシリーズ第1〜8回**。書籍は目次と該当本文を照合し、動画は公開説明欄・チャプターを参照した。「到達ライン」はMD-102の公式出題範囲に照らした練習目標。 [対象のPDF](https://chatgpt.com/library?file_id=libfile_317293fc40608191b517532206326e8d&account_id=personal)・[動画一覧](https://www.youtube.com/@%E7%A4%BE%E5%86%85%E3%83%8D%E3%83%83%E3%83%88%E3%83%8A%E3%83%93/videos)・[公式出題範囲](https://learn.microsoft.com/ja-jp/credentials/certifications/resources/study-guides/md-102)

## Intune本 目的から引くページ

表のページは紙面の番号。**今回のPDFでは紙面に12を足した番号**で開ける。

| 説明できるようにしたいこと | 紙面ページ（PDFの番号） | 到達ライン |
|---|---|---|
| **Intuneで何が変わるか** | **p.2–10**（14–22） | 端末の初期設定、アプリ展開、ポリシー、資産確認のうち、Intuneで担当できる作業を説明する。動画のIntune第1・2回と一緒に読む。 |
| **Entra参加とIntune登録** | **p.30–48、49–61**（42–60、61–73） | Registered／Joined／Hybrid Joinedをケースで選ぶ。自動登録のMDMユーザースコープ、登録制限、デバイスグループと登録後の確認まで。 |
| **AutopilotとESP** | **p.62–84**（74–96）。展開モードはp.64、ESP設定はp.75、デバイス準備はp.84 | ユーザー主導、事前プロビジョニング、自己展開から要件に合うものを選ぶ。ESPで何を待つか、どの段階で失敗したかを説明する。デバイス準備との比較は[現行公式表](https://learn.microsoft.com/en-us/autopilot/device-preparation/compare)も使う。 |
| **構成を配り、適用されない理由を調べる** | **p.140–145、177–180、276–284**（152–157、189–192、288–296）。カスタム／ADMXはp.185–209、GPO移行はp.285–296 | 設定カタログから設定を選び、ユーザー／デバイスへ割り当てる。フィルターと設定項目ごとの適用結果を確認する。OMA-URI、ADMX、GPO分析は使う場面を説明できるようにする。 |
| **端末の準拠を判定し、アクセスを制御する** | **p.538–555、577–584**（550–567、589–596）。カスタム準拠はp.556–565 | 準拠と非準拠の端末で期待する結果を決め、Intuneの非準拠項目とEntraのアクセス評価を追う。検証は[レポート専用の公式手順](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-all-users-device-compliance)を参照。 |
| **Win32アプリを配り、失敗を調べる** | **p.340–357、371–385、386–414**（352–369、383–397、398–426）。Store／M365 Appsはp.358–370 | 方式選定、サイレントコマンド、必要条件と検出、必須／使用可能、配信状況までつなぐ。特にp.353–354の検出・依存関係・置き換えと、p.408–414の診断を確認。 |
| **更新を段階展開する** | **p.224–246**（236–258）。Autopatchはp.239から | 更新リング、機能更新、品質更新で何を制御するかを言える。パイロット／本番の割り当てと結果の確認先を説明。Hotpatchは[現行公式](https://learn.microsoft.com/en-us/intune/device-updates/windows/configure-hotpatch)を補う。 |
| **端末を保護し、MDEのリスクを使う** | **p.247–275、566–576**（259–287、578–588） | BitLocker、Firewall、ASR、ベースライン、Defenderの違いをケースで説明。MDEの端末リスクをIntuneの準拠条件に使い、Entraがアクセスを評価する流れを追う。 |
| **iOS／Androidの登録と更新** | **p.86–138、298–300、331–337**（98–150、310–312、343–349） | Apple MDMプッシュ証明書、ABM／ADE、Managed Google PlayとAndroid Enterpriseを整理。社用・専用・仕事用プロファイルに応じて登録方式を選ぶ。 |
| **BYODとモバイルのアプリを管理する** | **p.433–460、468–476、490–505、585–593**（445–472、480–488、502–517、597–605）。Windowsの例はp.478–489 | Appleのアプリ配布、Managed Google Play、アプリ構成・保護の役割を説明。BYODで業務データのコピーなどを制御し、条件付きアクセスでアプリ保護を要求するケースを組み立てる。 |
| **運用で確認・修復・権限委譲する** | **p.212–223、596–612、625–632、634–640、658–683**（224–235、608–624、637–644、646–652、670–695） | 検出と修復の2本立て、レポートとEndpoint Analytics、Graphでの端末一覧、同期・リタイア・ワイプ、RBACとスコープ、テナント正常性を使い分ける。 |
| **Intune Suiteの位置づけ** | **p.181–184、338、641–657**（193–196、350、653–669）。EPMの紹介はp.12・659 | Cloud PKI、Tunnel for MAM、Remote Helpを要件から選ぶ。EPMのルールやEnterprise App Catalogの操作はMicrosoft Learnで具体化する。 |

## 社内ネットナビ どこを見て何を言えるようにするか

### Intuneシリーズ

| 動画 | 見る箇所 | 到達ライン |
|---|---|---|
| [第1回 Entra IDの次に学ぶべきIntuneとは？役割と全体像を解説](https://www.youtube.com/watch?v=3dv6s6wEWg8) | **6:40・7:33・9:20** | MDMは端末、MAMは対応アプリ内の業務データを管理。Entraのアクセス判定とIntuneの端末管理を分けて説明する。 |
| [第2回 Intuneで何ができる？PC・スマホ・アプリ・セキュリティを丸ごと管理](https://www.youtube.com/watch?v=HpHEAnURYfI) | **4:39・7:53・14:20** | 端末の設定統一、部署別アプリ配信、紛失・退職時の操作を、入社から返却までの流れで説明する。 |
| [第3回 Intuneを触る前に必ず要る土台｜デバイス登録3パターンを図解](https://www.youtube.com/watch?v=J8OBCemr91c) | **2:38・13:21・16:21** | 3方式はEntra上での端末の関係。会社PC・BYOD・既存AD環境の例で方式を選び、Intune登録がどこに加わるか説明する。 |
| [第4回 Windows Autopilotとは？PCキッティングを自動化する仕組みを初心者向けに解説](https://www.youtube.com/watch?v=-YSfzTdPhEQ) | **4:40・7:45・10:30** | 誰がPCを登録し、初回起動で何が起き、どの製品が何を担当するかを説明。モードとESPは本p.64・75、方式比較は現行公式で詰める。 |
| [第5回 300台のPC、1台ずつ見て回りますか？Intuneのコンプライアンスポリシー](https://www.youtube.com/watch?v=ZXlezTGTS2M) | **3:41・5:51・9:17** | 構成・準拠・アクセス制御の役割を区別。不適合の理由を見つけ、何を是正すると準拠に戻るか説明する。 |

### Entra IDシリーズ

| 動画 | 見る箇所 | 到達ライン |
|---|---|---|
| [第1回 Entra IDとは何か？](https://www.youtube.com/watch?v=8RDgL2mCwxY) | **0:50・4:21** | Entraが本人を確認し、アプリへのアクセスを扱うと説明できる。 |
| [第2回 Entra IDの仕組み｜サインイン・MFA・SSO・条件付きアクセス](https://www.youtube.com/watch?v=7Cy7HrGhTb4) | **0:47・5:25・7:17** | 認証、条件評価、アプリへのアクセスの流れをたどれる。MFAを求める例と、アクセスを拒否する例を言える。 |
| [第3回 オンプレADとの関係を理解する](https://www.youtube.com/watch?v=C4oIsvVL1EM) | **1:54・3:27** | ADとEntraを併用する理由を、既存ドメイン依存とクラウド利用を例に説明できる。 |
| [第4回 ADとEntra IDはどうやって連携してる？同期の仕組みをわかりやすく解説](https://www.youtube.com/watch?v=Jv4nbRWxMAI) | **3:02・4:59・7:31** | ディレクトリ同期と端末のHybrid Joinを区別する。PHSの目的と、クラウド側に何が同期されるかを概要で説明する。 |
| [第5回 Entra IDはで結局できることは何のか？](https://www.youtube.com/watch?v=r88gk-UkHA0) | **5:21・6:12** | 条件付きアクセスがIntuneの準拠情報を使う理由を、アプリにアクセスする場面で説明する。 |
| [第6回 なぜ1回のログインで全部使える？SSOの仕組みを初心者向けに完全解説](https://www.youtube.com/watch?v=ldjWA8ocPjU) | **3:39・7:46** | トークンやアサーションでSSOが成り立つ概要を説明する。SAML・OIDCはここでは役割と違いが言える深さで進む。 |
| [第7回 MFAとは？パスワードだけが危険な理由とパスワードレス認証を初心者向けに解説](https://www.youtube.com/watch?v=XplDwo0nfPs) | **8:20・10:48・15:14** | Windows Hello for Business、MFA、パスワードレスを使い分ける言葉を持つ。認証方法と、アクセス時にMFAを要求するポリシーの関係を説明する。 |
| [第8回 条件付きアクセスを正しく理解する｜Entra IDとIntune、役割を混同してない？](https://www.youtube.com/watch?v=0gkTY_E99rM) | **18:00–26:00、30:00** | Intune側で準拠条件、Entra側で対象と許可制御を決める。レポート専用で期待結果をたしかめ、本p.541・577から手順を再現する。 |

## 公式で補う箇所

- **新しめの試験項目**。プロビジョニングポリシー・イメージ・接続を選ぶ[Windows 365](https://learn.microsoft.com/en-us/training/modules/manage-windows-365/)、対象ファイルの昇格を設定する[EPM](https://learn.microsoft.com/en-us/intune/epm/overview)、カタログでアプリを管理する[Enterprise App Catalog](https://learn.microsoft.com/en-us/intune/app-management/deployment/enterprise-app-management)、端末のデータを読む[Device Query](https://learn.microsoft.com/en-us/intune/advanced-analytics/device-query)、検知や推奨を確認する[IntuneのSecurity Copilotエージェント](https://learn.microsoft.com/en-us/intune/agents/)。
- **本で扱いが薄い項目**。非Windowsは[macOSの登録](https://learn.microsoft.com/en-us/mem/intune-service/fundamentals/deployment-guide-enrollment-macos)と[Androidのzero-touch等の条件](https://learn.microsoft.com/en-us/intune/device-enrollment/android/ref-corporate-methods)を比較する。端末保護はアプリの実行を管理する[App Control for Business](https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-app-control-policy)、変更承認は[複数管理者承認](https://learn.microsoft.com/en-us/intune/fundamentals/role-based-access-control/multi-admin-approval)を確認する。
- **本の旧仕様**。p.477・534のWIPはWindows 11 24H2から削除。p.586の旧クライアントアプリ許可制御は2026年6月30日から読み取り専用へ移行したため、現行の演習はp.591のアプリ保護ポリシーを要求する方式で進める。 [Windowsの削除機能](https://learn.microsoft.com/en-us/windows/whats-new/removed-features)・[条件付きアクセスの移行案内](https://learn.microsoft.com/en-us/entra/identity/conditional-access/migrate-approved-client-app/)

## 最初に回す順番

**Intune第3回 → 第4回 → 第5回 → Entra第8回**。対応する本の **p.30–48 → p.62–84 → p.541–555・577–584** を開き、検証用PCの「登録」「構成と配信」「準拠とアクセス評価」をつなげる。最後に、経験のあるWin32配信を本p.353–354・408–414で確認し、未適用を1件説明できるところまで進める。
