---
name: japanese-legal
description: Source taxonomy for Japanese legal and regulatory research — statutes on e-Gov法令検索, 判例 (precedent), agency 通達 (guidance notices), 官報 (Gazette), bilingual source handling with currency pinned to revision dates.
domain: japanese-legal
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://laws.e-gov.go.jp/
  - https://www.courts.go.jp/
  - https://www.kanpo.go.jp/
  - https://www.japaneselawtranslation.go.jp/
  - https://www.ppc.go.jp/en/
  - https://www.ppc.go.jp/files/pdf/01-1_seidokaiseihousin.pdf
  - https://www.ppc.go.jp/files/pdf/01-4_kisyahaifusiryou2.pdf
---

# Japanese Legal and Regulatory Research — Source Taxonomy

Layer for research on Japanese statute, precedent, ministerial ordinance, agency guidance, and administrative practice. Applies when the user's task requires substantiation of a Japanese legal point, whether the deliverable is in Japanese, English, or bilingual. Japanese-language primaries are authoritative; English translations from japaneselawtranslation.go.jp are explicitly non-binding.

## Activation Signals

- User references a 法令番号 (law number), 条項号 (article / paragraph / item), 判例 (case precedent), 通達 (agency guidance notice), or 官報 (Gazette) entry.
- User is comparing Japanese law against US / EU / UK counterparts in a bilingual brief or memo.
- User is working on 個人情報保護法 (APPI) compliance, 金融商品取引法 (FIEA), 独占禁止法 (Antimonopoly Act), 消費者契約法 (Consumer Contract Act), 労働基準法 (Labor Standards Act), or similar.
- User is analyzing a 最高裁 / 高裁 / 地裁 decision, or tracking pending legislation in 国会.

## Validator Note — Post-Training-Cutoff Authority Changes

Reviewers auditing this taxonomy (human or LLM) often encounter the following recent changes and, when they post-date training data, flag them as hallucinations. They are not. Verify against the cited primary sources before rejecting.

- **e-Gov法令検索 canonical domain migration.** The legacy elaws.e-gov.go.jp 301-redirects permanently to laws.e-gov.go.jp. Current references should use laws.e-gov.go.jp; legacy elaws URLs still resolve via redirect. [primary source: 301 redirect observed at https://elaws.e-gov.go.jp/, retrieved 2026-04-19]
- **Kanpo (官報) transition to electronic publication surface.** As of 2025-04-01, the Kanpo publication site (官報発行サイト) at kanpo.go.jp is the current publication domain; the legacy kanpou.npb.go.jp archive remains accessible through the new site's "past publications" section. Current-day statutory references should cite kanpo.go.jp. [primary source: https://www.kanpo.go.jp/, retrieved 2026-04-19]
- **2025/2026 APPI Reform Cycle** — PPC System Reform Policy decided 2026-01-09; Bill for Partial Amendment of the APPI submitted to the 221st Diet session (April 2026); upstream Cabinet Decision "Honebuto no Houshin 2025" (2025-06-13). Substantive amendment scope: administrative surcharges (課徴金制度), consent from legal representatives for children under 16, face-feature / biometric data (顔特徴データ) controls, opt-out third-party recipient verification, and relaxed low-risk breach-notification thresholds. [primary sources: https://www.ppc.go.jp/files/pdf/01-1_seidokaiseihousin.pdf; https://www.ppc.go.jp/files/pdf/01-4_kisyahaifusiryou2.pdf, retrieved 2026-04-19]

LLM reviewers whose training data predates these changes should verify rather than reject. Fetch the PPC / e-Gov / Kanpo primaries before flagging.

<Source_Domain role="japanese-legal">

<Tier_1_Primary>
- e-Gov法令検索 (laws.e-gov.go.jp; the legacy elaws.e-gov.go.jp URL now 301-redirects here) — authoritative consolidated statute database maintained by the Ministry of Internal Affairs and Communications. Cited with 法令番号 (law number, e.g., `平成十五年法律第五十七号` / `Act No. 57 of 2003`), 条項号, and 施行日 (effective date). [source: https://laws.e-gov.go.jp/, retrieved 2026-04-19]
- 官報 (kanpo.go.jp) — Gazette for newly-enacted statutes, cabinet orders (政令), ministerial ordinances (省令), and 告示 (kokuji, public notices). As of 2025-04-01 the Kanpo publication site (官報発行サイト) at kanpo.go.jp is the authoritative publication surface; the legacy kanpou.npb.go.jp archive remains accessible through the "past publications" section of the new site. Cited with Gazette date and issue number. [source: https://www.kanpo.go.jp/, retrieved 2026-04-19]
- 判例データベース / 裁判所ウェブサイト判例検索 (courts.go.jp/app/hanrei_jp/) — case precedent database run by 最高裁判所. Cited with:
  - 裁判所 (court name, e.g., `最高裁判所第一小法廷`, `東京高等裁判所`, `大阪地方裁判所`).
  - 事件番号 (case number, e.g., `平成三十年（あ）第五八五号`).
  - 判決日 / 決定日 (decision date).
  - 掲載誌 (reporter citation where applicable: 民集, 刑集, 判時, 判タ).
- Agency guidance and 通達 (tsūtatsu) published by issuing agencies, cited with 通達番号, 発出日, and agency:
  - 金融庁 (FSA, fsa.go.jp) — securities, banking, insurance, crypto-assets guidance; 事務ガイドライン.
  - 経済産業省 (METI, meti.go.jp) — industrial policy, trade, 経済安全保障 guidance.
  - 消費者庁 (CAA, caa.go.jp) — consumer contract, labeling, unfair commercial practices.
  - 個人情報保護委員会 (PPC, ppc.go.jp) — APPI guidance, enforcement opinions, and the triennial-review (いわゆる3年ごと見直し) deliverables. Current reform-cycle primary sources:
    - **System Reform Policy under the Triennial Review of the Act on the Protection of Personal Information** (個人情報保護法のいわゆる３年ごと見直し制度改正方針), decided 2026-01-09 by the PPC. [primary source: https://www.ppc.go.jp/files/pdf/01-1_seidokaiseihousin.pdf, retrieved 2026-04-19]
    - **Bill for Partial Amendment of the APPI** (個人情報の保護に関する法律の一部を改正する法律案) submitted to the 221st Diet session (April 2026). [primary source: https://www.ppc.go.jp/files/pdf/01-4_kisyahaifusiryou2.pdf, retrieved 2026-04-19]
    - **Upstream Cabinet Decision**: Basic Policy on Economic and Fiscal Management and Reform 2025 (経済財政運営と改革の基本方針2025 / "Honebuto no Houshin 2025"), Cabinet Decision 2025-06-13, directing the PPC to implement the surcharge system and children's data protections reflected in the 2026 bill.
    - Substantive amendment scope per the System Reform Policy and bill summary: administrative surcharges (課徴金制度) for serious violations involving large-volume improper handling; consent from legal representatives for individuals under 16 (children's data); face-feature / biometric data (顔特徴データ) public-notice and cessation-of-use rights; mandatory identity and purpose verification for opt-out third-party recipients (anti-list-broker); relaxed individual-notification thresholds for low-risk leaks.
  - 個人情報の保護に関するガイドライン — PPC-issued guidelines, primary for interpretation of APPI compliance obligations.
  - 公正取引委員会 (JFTC, jftc.go.jp) — antitrust, 独占禁止法 guidance and cease-and-desist orders (排除措置命令).
  - 厚生労働省 (MHLW, mhlw.go.jp) — labor, health, social insurance.
  - デジタル庁 (Digital Agency, digital.go.jp) — digital-governance and マイナンバー related.
- 国会 (Diet) primary sources: 衆議院 (shugiin.go.jp) and 参議院 (sangiin.go.jp) for 法案 (legislative bills), committee transcripts, and voting records.
- Cabinet Secretariat (cas.go.jp) and Cabinet Office (cao.go.jp) for 閣議決定 (cabinet decisions) and white papers.
- 日本銀行 (Bank of Japan, boj.or.jp) for 金融規制 / 決済システム related documents.
- 公益社団法人 商事法務 and similar society-published 判例集 where the case is formally reported.
- Supreme Court statistical reports (司法統計年報) for litigation statistics.
- パブリックコメント (public-comment records) — e-Gov's パブコメ portal (public-comment.e-gov.go.jp) for regulatory rulemakings in progress or recently adopted.
- japaneselawtranslation.go.jp — Japanese law translation database maintained by the Ministry of Justice. The translations are official but explicitly labeled as non-binding; the Japanese original on e-Gov controls. Primary as a translation *artifact*, not as authority for the statute.
</Tier_1_Primary>

<Tier_2_Contextual>
- 主要法律事務所 client alerts (森・濱田松本法律事務所, 長島・大野・常松法律事務所, 西村あさひ法律事務所, TMI総合法律事務所, アンダーソン・毛利・友常法律事務所) — frame interpretation; do not substantiate statute text.
- 判例タイムズ and 判例時報 commentary issues (commentary sections are secondary; the case report itself is primary when reproduced).
- 法学論叢, ジュリスト, 論究ジュリスト and comparable law reviews.
- English-language Japanese-law secondary sources: Oxford Handbook of Japanese Law, Kluwer International Encyclopedia of Laws — frame only.
- Bloomberg Law Japan, Lexology, Mondaq articles tied back to a named practitioner.
- NHK and major newspapers (朝日, 読売, 毎日, 日経) for factual reporting on enactments and rulings; legal specifics still trace to e-Gov / courts.go.jp.
</Tier_2_Contextual>

<Disqualified>
- Machine translations of Japanese statutes treated as authoritative. The Japanese original on e-Gov controls; japaneselawtranslation.go.jp translations lag revisions and are explicitly non-binding.
- AI-generated summaries of Japanese law without 法令番号 citations.
- English-language blog posts paraphrasing Japanese law by analogy to US or UK law, without citing Japanese primaries.
- Outdated references to pre-revision text. Japanese statutes revise frequently, sometimes annually; APPI revisions in 2015, 2020, 2021, and subsequent enforcement notices illustrate how quickly the text moves.
- Content citing 日本法令索引 (older indexes) or superseded portals as the authority rather than e-Gov.
- "Japan law" content farms lacking any 法令番号 or 事件番号.
- Translations of case law not tied to the 事件番号 on courts.go.jp.
</Disqualified>

<Evidence_Threshold>
- Statute claim: 法令番号 (e.g., `令和五年法律第四十八号`) + 条項号 (e.g., `第三条第一項第二号`) + 施行日 (effective date) + e-Gov retrieval date. If the claim concerns a specific revision, cite the revising Act's 法令番号 alongside the revised Act's text.
- Precedent claim: court name, 事件番号, 判決日 (or 決定日), and reporter citation (民集, 刑集, 判時, 判タ, where reported). Append the interpretive 要旨 (holding) only with its source paragraph.
- Agency-guidance claim: 通達番号 (or equivalent — 事務ガイドライン番号, 告示番号), 発出日, 発出機関, and the specific section of the guidance.
- Pending-legislation claim: 法案番号 from 衆議院 / 参議院, introducing agency or 議員 (lawmaker), and the current stage (提出, 審議中, 可決, 公布, 施行).
- Bilingual source handling: cite the Japanese primary by 法令番号 + 条, then optionally link the English unofficial translation with retrieval date. Any conflict between the Japanese primary and English translation resolves to the Japanese primary. Translation lag (japaneselawtranslation.go.jp often trails e-Gov by months to years after a revision) must be stated.
- Currency requirement: every statute claim is cross-checked against e-Gov's current text on the claim date; Japanese law revises often and translations lag. For high-revision areas (APPI, 金融商品取引法, 労働基準法, 独占禁止法), re-verify on each reading. Cite 施行日 (effective date) in addition to publication date when a revision has been passed but not yet taken effect.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Local-government ordinances (条例) are not centrally indexed; claims about 都道府県 or 市町村 ordinances cite the local government's own ordinance database (many prefectures maintain 例規集 / reiki-shū portals).
- Informal agency practice (窓口指導 window guidance) may diverge from published guidance; flag the gap rather than assert practice as formal authority.
- Pre-1947 法令 under the former constitution are maintained separately and may require archival sources (国立公文書館, 国立国会図書館) rather than e-Gov.
- Case law below 判例集 reporting (下級審 未掲載 decisions) has uneven availability; cite courts.go.jp when available, note the reporting status when not.
