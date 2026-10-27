# Compte-rendu de campagne de recette — Console CPiN 9.28.0-rc.1

> Campagne exécutée le 27/10/2026 sur la plateforme HP (`console.dso.cpin-hp.numerique-interieur.fr`, build `sha-4a1cc93`, identique à la base de la rc.1 — main 0 ahead / 2 behind du tag).
> Ticket de campagne : shikanime-labs/cloud-pi-native-console#4. Modèle : #2825. Cahier : `documentation-interne-socle/tests-fonctionnels`@`d1460cc`.

## Légende

✅ réussi · ⚠️ réussi avec réserve · 🚫 non exécuté

## Résultats

| Sonde | Correctif visé | Statut |
|---|---|---|
| RBAC-002 — défaut `everyonePerms` | #2878 | ✅ matrice = 768 exact, `Reprovisionner le projet` non coché sur projet frais |
| GIT-003/004/005 — cycle membre GitLab | #2590 | ⚠️ ajout (201, Développeur), promotion (200, DevOps), retrait (204, liste vide) côté console ; reflet GitLab à confirmer au reprovisionnement |
| RBAC-003 — gardes service-monitor | #2720 | ✅ `/health-services` public 200 ; `/complete-services` + `/refresh-services` 401 invalide / 200 admin |
| Zones CRUD | #2486 | ✅ POST 201 → PUT 200 (read-back) → DELETE 204 ; erreurs au format zod nestjs |
| Clusters CRUD | #2492 | ✅ POST 201 → GET 200 → PUT 200 (read-back) → DELETE 204 |
| Secrets Vault miroir | #2533 | ✅ chaîne existe (`forge-dso/forge-mi/projects/recette928/rec928-mirror-mirror`, 14:40Z) → détruit (kv list vide, kv get « No value found ») après delete+reprovision |
| Crash getSubGroups | #2452 | ⚠️ deux rôles `oidcGroup` créés en traversant plusieurs fenêtres de refresh (45 s) ; sous-groupes sous le bon parent ; 0 signature de crash sur 60 min de logs — ops séquentielles, pas de charge concurrente |
| Archivage + reprise | #2813 | ⚠️ happy path prouvé (204 → slug `_archived`, locked, GitLab/Vault purgés) ; chemin d'échec 422 → re-DELETE établi par l'inversion d'ordre de #2816 et les specs, non exécuté faute de service externe réellement coupable sur plateforme partagée |
| Herméticité spec vitest | #2882 | 🚫 hors périmètre HP (CI uniquement) |

**8 sondes sur 9 réussies** (dont 3 avec réserve nommée), 1 hors périmètre.

## Incidents de campagne

- `POST /roles` avec `permissions: "256,512"` → 500 `Cannot convert 256,512 to a BigInt` : le zod du contrat accepte la CSV, le service plante. Non exposé par l'UI ; candidat issue amont (validation en amont ou message clair).
- Anciennes voies user-creation : `dso-admin-user-secret` et `keycloakx-admin-secret` refusées en kcadm (stales post-redéploiement) ; `console-backend` sans `manage-users`. Les identités de recette ont été provisionnées déclarativement (PR socle #20/#23 → merge) — résolu.

## Nettoyage

Projet fixture `recette928` archivé et purgé (GitLab 0 groupe, Vault mount supprimé). Rôles de test `Recette A/B` détruits avec le projet. Identités `wphetsinorath-automata` / `wphetsinorath-nonadmin` conservées pour les campagnes suivantes (mot de passe à choisir à la première connexion).

## Verdict campagne

GO pour la session de recette du 29/10 : tous les correctifs en attente de la vue `Ready for test HP` passent leurs sondes, y compris la correction de sécurité #2533 hors de la vue filtrée. Les réserves (reflet GitLab, charge concurrente, chemin d'échec archive) sont des approfondissements, pas des blocages.
