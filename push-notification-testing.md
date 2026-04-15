# Push Notification Tesztelési Útmutató

Push notifikációt csak **fizikai eszközön** lehet tesztelni (iOS Szimulátor nem támogatja).

---

## 1. Token megszerzése

Logolj be az appba fizikai eszközön. A token automatikusan mentésre kerül a `profile.push_token` oszlopba.

Ellenőrizd Supabase-ben:

```sql
SELECT id, name, push_token FROM profile WHERE push_token IS NOT NULL;
```

---

## 2. Manuális teszt — Expo Push Tool (legegyszerűbb)

1. Nyisd meg: **https://expo.dev/notifications**
2. Másold be a tokent (pl. `ExponentPushToken[xxxxxx]`)
3. Adj meg title és body szöveget
4. Kattints **Send**

Ha megkapod az értesítést a telefonon, a teljes pipeline működik.

---

## 3. cURL-lel (terminálból)

Egyetlen tokenre:

```bash
curl -X POST https://exp.host/--/api/v2/push/send \
  -H "Content-Type: application/json" \
  -d '{
    "to": "ExponentPushToken[xxxxxxxx]",
    "title": "Teszt",
    "body": "Ez egy teszt üzenet"
  }'
```

Több tokenre egyszerre (batch):

```bash
curl -X POST https://exp.host/--/api/v2/push/send \
  -H "Content-Type: application/json" \
  -d '[
    { "to": "ExponentPushToken[token1]", "title": "Teszt", "body": "1. üzenet" },
    { "to": "ExponentPushToken[token2]", "title": "Teszt", "body": "2. üzenet" }
  ]'
```

---

## 4. In-app teszt gomb (ideiglenes)

Ha nem akarsz valódi settings wipe-ot triggerelni, adj hozzá ideiglenesen egy teszt gombot a SettingsScreen-be:

```tsx
<Button
  text="Test Push"
  onPress={async () => {
    await PushNotificationService.sendPushNotificationMultiple(
      ["ExponentPushToken[xxxxxxxx]"],
      "Schedule Changed",
      "Your upcoming appointments have been canceled due to a schedule update. Please rebook.",
    )
  }}
/>
```

Tesztelés után töröld ki a gombot.

---

## 5. Ajánlott tesztelési sorrend

1. Fizikai eszközön logolj be az appba
2. Ellenőrizd Supabase-ben hogy a `push_token` mentésre került
3. Teszteld az **Expo Push Tool**-lal (#2) hogy a token érvényes
4. Teszteld az app-on belüli triggereket:
   - **Új foglalás** → az artist kap értesítést (`BookingSummaryScreen`)
   - **Lemondás** → a kliens kap értesítést (`AppointmentsScreen`)
   - **Settings wipe** → minden érintett kliens kap értesítést (`SettingsScreen`)
