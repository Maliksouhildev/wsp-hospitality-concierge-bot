# 💬 WSP Concierge — Production Chat History & Trace Scenarios

This document illustrates typical conversation traces handled autonomously by the n8n workflows and AI subagents.

---

## Scenario A: Guest Booking Flow (Algerian Darija)

**Customer**:  
> *"Salam Kho rani 7ab nekri partma lghedoua nchallah f Hydra pour 3 nuits, 2 personnes"*

**Bot Execution Trace**:
1. `WebhookReceiveMessage`: Meta webhook triggers with message text and customer phone.
2. `AiRouteur1`: Analyzes *"nekri" + "lghedoua" + "pour 3 nuits"*. Correctly classifies as `voyageur_potentiel`.
3. `BookingAgentVoyageur`: Calls `DisponibilitesHospitable` tool with parameters:
   - Check-in: Tomorrow
   - Nights: 3
   - Location: Hydra
   - Guests: 2
4. `wsp _ dispo`: Queries Hospitable calendar API, validates unreserved properties, calculates quote (DZD & EUR conversion via `ScrapeDevisesquare`).
5. `WaImageVoyageur`: Pulls high-resolution property photos from cache and dispatches to WhatsApp.

**Bot Response**:
> *"Wa alaykoum salam ! Bienvenue chez nos services concierge. 🌟*  
> *Pour demain à Hydra (3 nuits, 2 personnes), nous avons notre magnifique F2 'Le Panoramique Hydra' disponible.*  
> *✨ Équipements : Climatisation, Wifi haut débit, Place de parking privée, Cuisine équipée.*  
> *💵 Tarif total : 24,000 DZD pour les 3 nuits (ménage inclus).*  
> *(Photos envoyées ci-dessous 👇)*  
> *Souhaitez-vous que je bloque la réservation pour vous ?"*

---

## Scenario B: Midnight Arrival Flow (`wsp : voyageur minuit`)

**Customer (01:45 AM)**:  
> *"Salam khouya rani lha9t l'immeuble habit na3ref le code de la boîte à clés stp"*

**Bot Execution Trace**:
1. `IsTextMessage`: Webhook detects message timestamp past 00:00.
2. `wsp _ voyageur minuit`: Activates midnight protocol.
3. `RedisCheckReservation`: Verifies confirmed booking matching customer phone number.
4. Identifies apartment lockbox code and building entrance digicode.

**Bot Response**:
> *"Bonsoir et bienvenue à Alger ! 🌙*  
> *Voici vos instructions pour le check-in autonome en toute sérénité :*  
> *🔑 Code porte immeuble : 1489A*  
> *📦 Boîte à clés (à droite de la porte 2B) : Code 7291*  
> *📶 Wifi : 'Residence_Alger_5G' / Mdp : 'AlgerGuest2026'*  
> *En cas d'urgence la nuit, un concierge d'astreinte reste joignable. Bonne installation !"*

---

## Scenario C: Property Owner Lead Capture (`proprietaire_potentiel`)

**Customer**:  
> *"Bonjour, j'ai un F3 neuf à Sidi Yahia que je voudrais mettre en sous-location et gestion complète."*

**Bot Execution Trace**:
1. `AiRouteur1`: Detects `"j'ai un F3" + "gestion complète"`. Classifies as `proprietaire_potentiel`.
2. `BookingAgentProprietaire`: Initiates host qualification questionnaire.
3. `RedisCrmProprietaire`: Caches property details (location, rooms, state).
4. `GenererScriptGamma`: Prepares dynamic presentation deck and alerts agency admin.

**Bot Response**:
> *"Bonjour ! Enchanté. Nous serions ravis de valoriser votre F3 à Sidi Yahia avec notre service de conciergerie haut de gamme.*  
> *Nous gérons l'accueil des voyageurs, le ménage hôtelier, la literie et l'optimisation des revenus sur Airbnb, Booking.com et en direct.*  
> *Pour vous préparer une estimation de rentabilité : l'appartement est-il déjà meublé, et dispose-t-il d'un parking ?"*
