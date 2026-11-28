# CLAUDE.md - SafeCore-NetSec working guide

Personal portfolio project (Cisco Packet Tracer). User-facing restart steps (Korean) are in `작업_이어하기_가이드.md`. The user attends a network class; each class day the instructor gives a daily work plan (작업 계획서) written for the instructor's own topology. The job is to re-map that plan onto THIS SafeCore topology and execute it step by step with the user. Read this whole file before the first reply of a session.

## 1. Daily procedure (day N)

1. Read the latest state: this file, `README.md`, `03_설계도/네트워크_설계서.md`, the previous day's `02_작업 계획/` plan and `04_학습문서/` doc.
2. Study the instructor's plan. Translate EVERYTHING to SafeCore: device names, ports, addresses, AND the story (company names, servers, business reason must make sense for SafeCore, a firewall/AV/EDR vendor). Never carry the instructor's names (e.g. 새봄상사, 한결물류, HQ-R1) into portfolio docs.
3. **The instructor's required items are mandatory and come first.** Deeper work (attack reproduction, extra verification) is optional and only after the required items are done. Propose a depth table (required / optional deeper) with a recommendation and reasons; agree with the user before configuring.
4. Before relying on a command, probe support with `?` (see section 5). Split probe blocks at every mode change.
5. Execute one step at a time. For every step: full per-device command block, verify with `show`, record the result in the 학습문서 immediately, tell the user exactly when to capture and the file name.
6. Finish: 작업계획서 "6. 결과", 네트워크 설계서, 라우팅 비교 (if routing), README (핵심 성과, 보안 시나리오, 일차별 기록), commit; push only after the user confirms.
7. Work that might break things is done on a git branch (day 6 `ospf`, day 7 `eigrp`), fast-forwarded into main at the end. `.pkt` is binary and cannot be merged.

## 2. Communication rules (user-enforced)

- Korean, formal 존댓말 (합쇼체) in chat and in user-facing docs. This file stays in English.
- Never write the tilde character (renders as strikethrough). Use "에서", "-", "부터 까지". No hanja, no emoji.
- Before every command block: device hostname AND the prompt to type at, e.g. **BW-R1**, `BW-R1#`에서 시작, 끝나면 다시 `BW-R1#`. Say which mode each line lands in when modes change.
- Never abbreviate per-device blocks ("위와 동일" forbidden). When something failed, re-issue the whole block for that step.
- Strategy/scope questions: propose a recommendation with reasons; do not bounce the question back.
- Verify facts with CLI (`show`, `?`, `show cdp neighbors`) instead of guessing or GUI hovering. Mark unverified things 미검증/미확인/추정.
- **Never give risky commands**: no `no router <proto>` on a live process, `reload`, `erase`, bulk deletes. On day 7 `no router eigrp 20` crashed Packet Tracer. If Packet Tracer state gets stuck, the safe fix is: save, close, reopen the `.pkt`.
- **Remind the user to save** (`Ctrl+S` in Packet Tracer) before risky steps and after each verified step. Do not save while an attack/broken test state is active.
- Do not doubt the user's captures; if output looks old, explain the likely cause (e.g. command typed in another device's window).
- Unsupported commands: record as text only, do not ask for captures of errors.

## 3. Document rules

- 작업계획서 (portfolio): 날짜, 브랜치, 토폴로지, 1 목표, 2 작업 대상과 설정값 (one table), 3 작업 내용 (numbered), 4 확인 방법 (table), 5 되돌리는 방법, 6 결과. Short; no constraint tables or rationale paragraphs (one clause max). No mention of the instructor.
- 학습문서 (personal study): per step 목적 (one line), command block, short "왜 넣는가" table only for new commands, 결과 as dense bullets, 막힌 곳 as a compact table if needed. No repeated concepts from earlier days, no trivial noise. End with a short 면접 대비.
- Images in docs: `<img src="../99_첨부파일/..." width="480">` (one-line outputs 320, topology 760), each image on its own paragraph with blank lines around, grouped at the end of the step's results.
- Captures: `99_첨부파일/N일차_번호_설명.png`. The user pastes screenshots in chat; copy them from the chat image cache into `99_첨부파일` with the agreed names.
- Key values never in docs (`<키>`), passwords as `<실습용 비밀번호>`. Quote requirements only from `01_요구사항/프로젝트_계획서.md`.
- Only measured claims in README; keep 한계와 다음 단계 honest.

## 4. Current state (end of day 7, 2026-11-28)

- Branch main = latest. Devices: SC-HQ-R1 (ISR4331), SC-BR-R1 (2901), switches SC-SW-RND/QA/TS/ADM/SRV/TSBR, acquired company BlueWatch: BW-R1 (2901 + HWIC-2T), BW-SW1, BW-PC1.
- Addressing: VLAN 10 R&D, 20 QA, 30 TS-HQ, 40 ADM, 50 ServerFarm, 60 TS-BR = 192.168.X.0/24 (gw .1, switch mgmt .254). Main WAN 192.168.70.0/24 (HQ Vlan70 via Gi0/1/3, BR Gi0/1). Backup serial 192.168.80.0/30 (HQ Se0/2/0 DCE, BR Se0/3/0). Loopbacks HQ 192.168.255.1, BR 192.168.255.2, BR warehouse Lo1 192.168.90.33/27. BlueWatch LAN 172.16.10.0/24, HQ-BW link 172.16.20.0/30 (HQ Se0/2/1 DCE .1, BW-R1 Se0/3/0 .2). File server 192.168.50.11 (farm PCs .11-.14). R&D test PC 192.168.10.11, ADM PC 192.168.40.11, BR PC 192.168.60.11.
- Routing: OSPF 1 area 0 (HQ, BR), `passive-interface default` with only WAN links open, interface MD5 on all four WAN interfaces; cost Ethernet 1, serial 64. EIGRP 20 between HQ and BW-R1 (`no auto-summary`, MD5 key chain BW-KEY). HQ: `redistribute eigrp 20 subnets` into OSPF; toward BlueWatch only `network 192.168.50.0 0.0.0.255` (Vlan50 passive), no OSPF->EIGRP redistribution. ACL 110 out on HQ Se0/2/1 (permit eigrp, permit ip any 172.16.10.0/24, deny ip any any). BW-R1 Loopback50 (hijack test, 192.168.50.126/25) is shutdown.
- Free ports: HQ Gi0/0/2 cannot take copper in this Packet Tracer (not in cable list); HQ has no free NIM slot. BR Se0/3/1 free, BR HWIC-4ESW unused.
- Security gaps (planned): R&D isolation ACL, BlueWatch per-server ACL (they can reach the whole server farm /24), SSH instead of telnet, passwords on BR and BlueWatch devices, portal NAT, R&D VPN.

## 5. Packet Tracer limits verified in this build

- Not supported: `ip prefix-list`, `route-map`, `distribute-list` (EIGRP, OSPF, RIP), `distance <ad> <src> <wc> <acl>` (no ACL arg), RIP authentication, RIP `neighbor`, `clear ip eigrp` (2901), `show running-config interface X` (ISR4331), `debug ip ospf hello`, 2960 `switchport trunk encapsulation`.
- Quirks: after `no redistribute ospf 1` HQ kept "via Redistributed" EIGRP entries (fixed by reopening the file); OSPF redistribution of EIGRP is computed only at file load (new or lost EIGRP routes do not update BR's `O E2`); `clear ip ospf process` needs literal `yes`; loopbacks need `ip ospf network point-to-point` to advertise their mask; NIM-2T DCE has `clock rate 2000000` by default; NIM-ES2-4 ports need VLAN + SVI.
- Supported and used: EIGRP, OSPF, MD5 (OSPF interface, EIGRP key chain), `redistribute ... metric/subnets`, numbered ACL + `ip access-group`, `show ip eigrp topology`, `traceroute`.
