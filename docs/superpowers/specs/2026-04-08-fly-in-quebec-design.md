# Fly-in Québec — Design Document

**Date :** 2026-04-08
**Statut :** En validation
**Auteur :** Seb + Claude

---

## 1. Contexte & Opportunité

### Le problème
Les travailleurs des grandes villes du Québec qui s'intéressent au travail Fly-in/Fly-out (FIFO) dans les mines du Nord n'ont aucune ressource centralisée, francophone et spécialisée pour :
- Comprendre les conditions réelles de travail (camps, rotations, transport)
- Suivre les opportunités d'emploi en temps réel
- Évaluer les employeurs et les sites miniers
- Planifier leur transition vers le FIFO

### Le marché
- **27 800 travailleurs miniers** au Québec (2022-2024)
- **5 000-8 000 en rotation FIFO** (100% du Nord-du-Québec)
- **14 358 postes à combler d'ici 2028** (pénurie critique)
- Régions : Abitibi-Témiscamingue, Côte-Nord, Nord-du-Québec
- Salaire moyen : $80K-$120K/an

### Principaux employeurs FIFO
| Employeur | Mine/Site | Localisation | ~Effectif |
|-----------|-----------|-------------|-----------|
| Glencore | Raglan (nickel) | Nunavik | 1 280 |
| ArcelorMittal | Mont-Wright, Fire Lake | Fermont | 2 900 |
| Agnico Eagle | Canadian Malartic, LaRonde | Abitibi | 2 000+ |
| Champion Iron | Bloom Lake | Fermont | 450+ |
| Dhilmar (ex-Newmont) | Éléonore (or) | Baie-James | ~600 |
| Gold Fields | Windfall (or) | Baie-James | En construction |
| Rio Tinto | IOC | Labrador City | 1 800 |

### Concurrence
**Aucun concurrent direct.** Ce qui existe :
- **Explore les Mines** (explorelesmines.com) — Site gouvernemental gratuit, informatif mais générique
- **emploisdanslesmines.com** — Job board basique, pas de contenu éditorial
- **AEMQ** — Association d'exploration minière, pas orienté travailleurs
- **Indeed/LinkedIn** — Génériques, pas spécialisés FIFO

**En Australie** (benchmark mondial), aucun modèle d'abonnement travailleur n'a réussi. Les revenus viennent des employeurs et des services ponctuels.

---

## 2. Vision Produit

### Nom
**Fly-in Québec**

### Positionnement
Le premier portail francophone dédié au travail Fly-in/Fly-out dans les mines du Nord du Québec. Information, veille, communauté.

### Tagline
*"Tout ce que vous devez savoir avant de monter dans l'avion."*

### Audiences
- **Primaire :** Travailleurs des grandes villes (Montréal, Québec, Saguenay, Rouyn-Noranda) qui envisagent ou pratiquent le FIFO minier
- **Secondaire :** Employeurs miniers en pénurie de main-d'œuvre (14 358 postes à combler)

### Différenciation
1. **Moat francophone** — Zéro plateforme FIFO francophone existe
2. **Veille autonome IA** — Contenu frais sans effort éditorial constant
3. **Reviews de camps** — Information que personne n'agrège (comme Glassdoor pour les camps)
4. **Ton humain et pratique** — Pas corporatif, pas gouvernemental

---

## 3. Architecture du Contenu (3 Phases)

### Phase 1 — Le Média (Mois 1-3, MVP)

**Contenu gratuit (acquisition d'audience) :**
- **Guide des employeurs** — Fiches détaillées des 15-20 mines FIFO : localisation, rotation, salaires, transport, conditions
- **Guide des métiers** — 10-15 métiers les plus demandés avec formations, salaires, perspectives
- **Blog/actualités** — Articles SEO, conseils pratiques, témoignages
- **FAQ exhaustive** — Déductions fiscales nordiques, assurances, vie de camp, impact familial

**Contenu de veille autonome :**
- **Fil d'offres d'emploi agrégées** — Scraping quotidien d'Indeed, emploisdanslesmines, sites corporatifs, LinkedIn
- **Alertes nouveaux projets** — Monitoring des permis miniers, décisions BAPE, annonces gouvernementales
- **Tracker des conditions** — Mises à jour sur rotations, transport, hébergement par mine

### Phase 2 — Les Services B2B (Mois 4-8)
- **Affichage de postes employeurs** ($299/poste)
- **Profils employeurs premium** ($699/mois) — Page enrichie, mise en avant, analytics
- **Reviews de camps** — Système de reviews vérifiées
- **Newsletter sponsorisée** ($500-1000/envoi)

### Phase 3 — Le Hub (Mois 9-18)
- **Communauté/forum** entre travailleurs FIFO
- **Services CV spécialisés** ($299-599/CV)
- **Guides payants** ($29-49)
- **Outil de rotation** — Calendrier FIFO, countdown
- **Section famille** — Ressources pour conjoints/familles
- **Partenariats formations** (commission 10-15%)

---

## 4. Système de Veille Autonome

### Architecture

```
SOURCES                    PIPELINE               TRAITEMENT           PUBLICATION
─────────                  ────────               ──────────           ───────────
Job boards ──┐                                   
Sites corpo ─┤         Scrapers              Claude API           Site web (SSG)
Docs publics ┼───────► Playwright  ────────► Classification  ───► Newsletter
Réseaux soc. ┤         + Cron jobs           Extraction          Réseaux sociaux
Permis/BAPE ─┘         + BullMQ              Dédup + Résumé
                                             Enrichissement
```

### Sources et fréquences
| Source | Fréquence | Méthode |
|--------|-----------|---------|
| Indeed Québec mining | 6h | Scraping Playwright |
| emploisdanslesmines.com | 6h | Scraping |
| Sites carrières corporatifs (15+) | 12h | Scraping |
| LinkedIn jobs mining Québec | 12h | API/Scraping |
| BAPE, MERN (documents publics) | Hebdomadaire | Scraping |
| Permis miniers, annonces gouv. | Quotidien | Scraping |
| Groupes Facebook FIFO | 12h | Scraping |
| Reddit r/mining, r/quebec | 12h | API Reddit |

### Traitement IA (Claude API)
Pour chaque élément scrapé :
1. **Classification** : emploi / projet / actualité / condition
2. **Extraction structurée** : salaire, rotation, localisation, métier, employeur, camp
3. **Dédoublonnage sémantique** : éviter les doublons entre sources
4. **Résumé et rédaction** : français québécois naturel, ton informatif
5. **Enrichissement** : lien vers fiche employeur, conditions connues du camp, historique

### Détection de changements
- Hash du contenu pour détecter les modifications
- Diff sémantique pour les mises à jour partielles
- Alertes automatiques pour les nouvelles mines / fermetures / changements majeurs

---

## 5. Stack Technique

### Frontend
- **Next.js 15** (App Router, SSG + ISR pour le SEO)
- **Tailwind CSS + shadcn/ui**
- **Hébergement : Vercel**
- **Langue : FR prioritaire** (EN éventuel en Phase 3)

### Backend
- **Supabase** (PostgreSQL + Auth + Storage + Edge Functions)
- **API Routes Next.js** pour logique métier

### Services de Veille
- **Workers Node.js** avec scrapers Playwright
- **Hébergement : Fly.io** (ou Railway)
- **BullMQ** pour la queue de jobs de scraping
- **Redis** pour cache et déduplication
- **Claude API** pour traitement IA

### Schéma de données (tables principales)
```
employers          -- Fiches employeurs/mines
├── id, name, slug, description, location, rotation_type
├── transport_info, camp_conditions, salary_range
└── logo_url, website, careers_url

job_listings       -- Offres d'emploi agrégées
├── id, employer_id, title, description_raw, description_ai
├── salary_min, salary_max, rotation, location
├── source_url, source_name, scraped_at, expires_at
└── trade_category, certification_required

camp_reviews       -- Reviews de camps (Phase 2)
├── id, employer_id, rating, content
├── food_rating, internet_rating, room_rating, recreation_rating
└── author_hash (anonyme), verified, created_at

articles           -- Blog + contenu généré
├── id, title, slug, content, excerpt
├── category, tags[], published_at
└── source_type (editorial / ai_generated / hybrid)

projects           -- Projets miniers (tracker)
├── id, name, employer_id, status, mineral_type
├── location, investment_amount, jobs_estimated
├── start_date, production_date
└── bape_status, permit_status

scrape_jobs        -- Historique de scraping
├── id, source, status, items_found, items_new
└── started_at, completed_at, error_log

newsletter_subscribers
├── id, email, preferences (métiers, régions, fréquence)
└── created_at, confirmed
```

---

## 6. Modèle d'Affaires

### Sources de revenus par phase

| Phase | Source | Prix | Objectif mensuel | Revenu mensuel |
|-------|--------|------|-----------------|----------------|
| **Phase 2** | Affichage postes | $299/poste | 5-15 postes | $1 500-4 500 |
| **Phase 2** | Profil employeur premium | $699/mois | 2-5 employeurs | $1 400-3 500 |
| **Phase 2** | Newsletter sponsorisée | $500-1000/envoi | 2-4/mois | $1 000-4 000 |
| **Phase 3** | CV spécialisés FIFO | $299-599/CV | 10-20/mois | $3 000-12 000 |
| **Phase 3** | Guides payants | $29-49 | 30-50/mois | $900-2 500 |
| **Phase 3** | Commissions formations | 10-15% | Variable | $500-2 000 |

### Projections conservatrices

| Période | Revenus mensuels | Coûts mensuels | Profit |
|---------|-----------------|----------------|--------|
| Mois 1-3 | $0 | $150-300 | -$300 |
| Mois 4-8 | $2 000-8 000 | $300-500 | $1 500-7 500 |
| Mois 9-12 | $5 000-15 000 | $500-1 000 | $4 000-14 000 |
| Mois 13-18 | $10 000-25 000 | $1 000-2 000 | $9 000-23 000 |

### Coûts opérationnels
| Service | Coût/mois |
|---------|-----------|
| Vercel (hosting) | $0-20 |
| Supabase (DB + auth) | $0-25 |
| Fly.io (scrapers) | $5-15 |
| Claude API (traitement IA) | $30-80 |
| Domain + email | $15 |
| Outils marketing (Mailchimp, etc.) | $0-50 |
| **Total Phase 1** | **$50-205** |

### Seuil de rentabilité
~4-5 affichages de postes/mois OU 2 employeurs premium = breakeven

---

## 7. Analyse de Risques

| Risque | Prob. | Impact | Mitigation |
|--------|-------|--------|------------|
| Employeurs refusent de payer | Moyenne | Élevé | Audience d'abord, prouver valeur avec analytics |
| Scraping bloqué | Élevée | Moyen | Sources multiples, RSS fallback, partenariats |
| Marché trop petit | Faible | Élevé | 14K postes = demande réelle côté employeur |
| Concurrent copie | Faible | Moyen | Avantage premier arrivé + profondeur contenu |
| Contenu IA médiocre | Moyenne | Élevé | Review humain les premiers mois, prompts solides |
| Ralentissement minier | Faible | Élevé | Stratégie minéraux critiques du QC = tailwind |
| Problèmes légaux scraping | Moyenne | Moyen | Respecter robots.txt, termes d'utilisation |

---

## 8. Facteurs de Succès

1. **SEO agressif dès le jour 1** — "travail fly-in fly-out québec", "emploi mine nord québec", "vie camp minier" = mots-clés à faible compétition
2. **Contenu de qualité > quantité** — Les fiches employeurs doivent être meilleures que tout ce qui existe
3. **Reviews de camps = killer feature** — Info introuvable ailleurs, viralité naturelle
4. **Newsletter comme canal principal** — Les travailleurs FIFO lisent leurs emails en camp
5. **Réseau indirect à exploiter** — Témoignages, interviews, crédibilité par association
6. **Approche progressive** — Ne pas tout construire d'un coup, valider chaque phase

---

## 9. Indicateurs Clés (KPIs)

### Phase 1
- Visiteurs uniques/mois (objectif : 5 000)
- Abonnés newsletter (objectif : 500)
- Fiches employeurs publiées (objectif : 15-20)
- Articles publiés (objectif : 30+)

### Phase 2
- Employeurs payants (objectif : 5-10)
- Postes affichés/mois (objectif : 10-15)
- Reviews de camps soumises (objectif : 50+)
- Taux de conversion visiteur → newsletter (objectif : 5%+)

### Phase 3
- Revenus mensuels (objectif : $10K+)
- Membres communauté active (objectif : 500+)
- CV vendus/mois (objectif : 15+)
- NPS travailleurs (objectif : 40+)
