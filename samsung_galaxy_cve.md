# Samsung Galaxy CVE - Project Zero

> Liste exhaustive des vulnérabilités découvertes par Google Project Zero affectant les produits Samsung Galaxy

---

## 1. Vulnérabilités Qmage Codec (2020)

### CVE-2020-8899 (SVE-2020-16747)
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Critique |
| **CVSS** | 10.0 |
| **Composant** | libhwui.so (Qmage codec dans Skia) |
| **Type** | Buffer Overflow / Heap Corruption |
| **Vecteur** | Zero-click via MMS |
| **Découvreur** | Mateusz Jurczyk (Project Zero) |
| **Date rapport** | Janvier 2020 |
| **Patch** | Mai 2020 |

**Appareils affectés:** Tous les Samsung Galaxy depuis 2014 (Android 4.4.4+)
- Galaxy S5, S6, S7, S8, S9, S10
- Galaxy Note 3, 4, 5, 8, 9, 10
- Galaxy A series, M series, J series

**Description technique:**
- Overflow de buffer dans la fonction `QmageDecCommon_MakeColorTable_Rev8253_140615`
- 5,218 crashs uniques identifiés lors du fuzzing
- Exploitation via envoi d'image .qmg malformée par MMS
- Bypass ASLR nécessite 50-300 messages MMS (~100 minutes)
- Aucune interaction utilisateur requise

**Références:**
- https://project-zero.issues.chromium.org/issues/42451103
- https://projectzero.google/2020/08/mms-exploit-part-5-defeating-aslr-getting-rce.html

---

## 2. Exploit Chain In-the-Wild (2021)

### CVE-2021-25337
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **Composant** | Samsung Clipboard Provider |
| **Type** | Arbitrary File Read/Write |
| **Vecteur** | Application malveillante |
| **Patch** | Mars 2021 |

**Description:** Manque de contrôle d'accès dans le provider clipboard Samsung permettant lecture/écriture de fichiers arbitraires en tant qu'utilisateur système.

### CVE-2021-25369
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **Composant** | sec_log (Kernel) |
| **Type** | Information Leak |
| **Impact** | Bypass KASLR |
| **Patch** | Mars 2021 |

**Description:** Fuite d'information kernel permettant de récupérer les adresses de `task_struct` et `sys_call_table` pour contourner KASLR.

### CVE-2021-25370
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **Composant** | DECON driver (DPU) |
| **Type** | Use-After-Free |
| **Impact** | Kernel Code Execution |
| **Patch** | Mars 2021 |

**Description:** UAF dans le driver Display and Enhancement Controller permettant l'élévation de privilèges au niveau kernel.

**Références:**
- https://projectzero.google/2022/11/a-very-powerful-clipboard-samsung-in-the-wild-exploit-chain.html

---

## 3. Samsung Exynos Baseband - 18 Zero-Days (2023)

### Vulnérabilités Critiques (RCE Internet-to-Baseband)

#### CVE-2023-24033
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Critique |
| **CVSS** | 9.8 |
| **Composant** | Exynos Modem (SDP Parser) |
| **Type** | Memory Corruption |
| **Vecteur** | Réseau (sans interaction) |
| **Patch** | Mars 2023 |

**Description:** Le logiciel baseband ne vérifie pas correctement les types de format de l'attribut accept-type spécifié par SDP, conduisant à un déni de service ou exécution de code.

#### CVE-2023-26496
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Critique |
| **CVSS** | 9.8 |
| **Composant** | Exynos Modem |
| **Type** | Memory Corruption |
| **Vecteur** | Réseau (sans interaction) |

#### CVE-2023-26497
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Critique |
| **CVSS** | 9.8 |
| **Composant** | Exynos Modem |
| **Type** | Memory Corruption |
| **Vecteur** | Réseau (sans interaction) |

#### CVE-2023-26498
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Critique |
| **CVSS** | 9.8 |
| **Composant** | Exynos Modem |
| **Type** | Memory Corruption |
| **Vecteur** | Réseau (sans interaction) |

### Vulnérabilités Secondaires (Accès local/opérateur requis)

| CVE | Composant | Type |
|-----|-----------|------|
| CVE-2023-26072 | 5G MM Message Codec | Heap Buffer Overflow |
| CVE-2023-26073 | Exynos Modem | Memory Corruption |
| CVE-2023-26074 | Exynos Modem | Memory Corruption |
| CVE-2023-26075 | Exynos Modem | Memory Corruption |
| CVE-2023-26076 | Exynos Modem | Memory Corruption |
| + 9 CVE non assignées | Exynos Modem | Diverses |

**Appareils affectés:**
- Samsung Galaxy S22, M33, M13, M12, A71, A53, A33, A21s, A13, A12, A04
- Google Pixel 6, Pixel 7
- Vivo S16, S15, S6, X70, X60, X30
- Véhicules avec Exynos Auto T5123

**Mitigation:** Désactiver Wi-Fi Calling et VoLTE

**Références:**
- https://projectzero.google/2023/03/multiple-internet-to-baseband-remote-rce.html
- https://semiconductor.samsung.com/support/quality-support/product-security-updates/cve-2023-24033/

---

## 4. Vulnérabilités libsaped.so (2024)

### CVE-2024-49415
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **CVSS** | 8.1 |
| **Composant** | libsaped.so (C2 media service) |
| **Type** | Out-of-Bounds Write |
| **Vecteur** | Zero-click via RCS |
| **Découvreur** | Natalie Silvanovich (Project Zero) |
| **Patch** | Décembre 2024 |

**Appareils affectés:** Galaxy S23, S24 (avec RCS activé par défaut)

**Description technique:**
- La fonction `saped_rec` écrit dans un dmabuf de taille fixe 0x120000
- Avec `bytes_per_sample = 24`, peut écrire jusqu'à `3 * blocksperframe` bytes
- Un fichier APE avec grande valeur `blocksperframe` cause un overflow
- Exploitable via message audio RCS sans interaction utilisateur

### CVE-2024-49413
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **CVSS** | 7.1 |
| **Composant** | SmartSwitch |
| **Type** | Improper Cryptographic Signature Verification |
| **Impact** | Installation d'applications malveillantes |
| **Patch** | Décembre 2024 |

---

## 5. Vulnérabilités libimagecodec.quram.so (2025)

### CVE-2025-21042
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **CVSS** | 8.8 |
| **Composant** | libimagecodec.quram.so |
| **Type** | Out-of-Bounds Write |
| **Vecteur** | Zero-click via image (WhatsApp DNG) |
| **Exploitation** | Spyware LANDFALL (Moyen-Orient) |
| **Patch** | Avril 2025 |

**Appareils affectés:** Galaxy S22, S23, S24, Z Fold 4, Z Flip 4

**Description:** Vulnérabilité dans la bibliothèque de traitement d'images Quram, exploitée activement depuis juillet 2024 pour déployer le spyware commercial LANDFALL.

### CVE-2025-21043
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **CVSS** | 8.8 |
| **Composant** | libimagecodec.quram.so |
| **Type** | Out-of-Bounds Write |
| **Exploitation** | In-the-wild |
| **Patch** | Septembre 2025 |

**Appareils affectés:** Android 13, 14, 15, 16

---

## 6. Autres CVE Affectant Samsung Galaxy

### CVE-2019-2215
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **CVSS** | 7.8 |
| **Composant** | Android Kernel (Binder) |
| **Type** | Use-After-Free |
| **Vecteur** | Chrome Sandbox Escape |
| **Patch** | Octobre 2019 |

**Appareils affectés:** Galaxy S7, S8, S9, Pixel, Huawei P20, Xiaomi

### CVE-2024-44068
| Attribut | Valeur |
|----------|--------|
| **Sévérité** | Haute |
| **CVSS** | 8.1 |
| **Composant** | Samsung Mobile Processor |
| **Type** | Use-After-Free |
| **Impact** | Privilege Escalation |
| **Exploitation** | In-the-wild |
| **Patch** | Octobre 2024 |

---

## Résumé Statistique

| Catégorie | Nombre CVE | Plus Critique |
|-----------|------------|---------------|
| Codec Image (Qmage/Quram) | 4 | CVE-2020-8899 (CVSS 10.0) |
| Baseband Exynos | 18 | CVE-2023-24033 (CVSS 9.8) |
| Exploit Chain In-the-Wild | 3 | CVE-2021-25337 |
| Audio/RCS | 2 | CVE-2024-49415 (CVSS 8.1) |
| Kernel/Driver | 3 | CVE-2019-2215 |
| **TOTAL** | **~30** | |

---

## Chronologie

| Année | CVE Majeures | Événement |
|-------|--------------|-----------|
| 2019 | CVE-2019-2215 | Kernel UAF affectant multiples Android |
| 2020 | CVE-2020-8899 | Qmage MMS 0-click (tous Galaxy depuis 2014) |
| 2021 | CVE-2021-25337/25369/25370 | Exploit chain in-the-wild |
| 2023 | CVE-2023-24033 + 17 autres | 18 zero-days Exynos baseband |
| 2024 | CVE-2024-49415, CVE-2024-44068 | Zero-click RCS, processor UAF |
| 2025 | CVE-2025-21042, CVE-2025-21043 | Quram image codec (LANDFALL spyware) |

---

## Sources

- [Project Zero Issue Tracker](https://project-zero.issues.chromium.org/)
- [Project Zero Blog](https://projectzero.google/)
- [Samsung Security Updates](https://security.samsungmobile.com/)
- [Samsung Semiconductor Security](https://semiconductor.samsung.com/support/quality-support/product-security-updates/)
- [CISA KEV Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

---

*Dernière mise à jour: Janvier 2026*
