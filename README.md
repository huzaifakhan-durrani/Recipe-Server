# Recipe Book: recipe server

Everything the app downloads lives here: `recipes.json` and the pictures in `images/`.
Push to `main` and the app picks it up at its next check (once a day, on Wi-Fi, or from Settings > Check for updates).

```
recipes.json      the recipes, the categories and the optional "update the app" notice
images/           one picture per recipe, referenced from recipes.json
```

The app reads the file from `https://raw.githubusercontent.com/huzaifakhan-durrani/Recipe-Server/main/recipes.json`.

## The one rule: raise `contentVersion`

The app only installs a file whose `contentVersion` is higher than the one it already has.
**Raise it by one every time you change a recipe or a picture.** (The `appUpdate` notice is the exception: it is read on every check.)

## Adding a recipe

Add an object to `recipes`. Copy an existing one and change it. Required: `id` (lowercase letters, digits and `-`), `title`, `category` or `categories`, `difficulty` (`Easy`, `Medium`, `Hard`), `prepTime` and `cookTime` in minutes, `servings`, `ingredients`, `steps`.

```json
{
  "id": "chicken-tikka",
  "title": "Chicken Tikka",
  "description": "One or two sentences.",
  "category": "Dinner",
  "categories": ["dinner", "pakistani"],
  "difficulty": "Medium",
  "prepTime": 20,
  "cookTime": 20,
  "servings": 4,
  "image": "images/chicken_tikka.webp",
  "ingredients": [
    { "name": "Chicken thighs, cubed", "quantity": 700, "unit": "g", "group": "MEAT" }
  ],
  "steps": [
    { "text": "Marinate.", "tip": "Optional hint.", "timerMinutes": 120 },
    { "text": "A step without a timer." }
  ],
  "tags": ["Grilled"],
  "popular": 90,
  "version": 1
}
```

- `group` is `PRODUCE`, `MEAT`, `DAIRY`, `PANTRY` or `SPICES` (it decides the aisle on the shopping list).
- Use plain units: `g`, `kg`, `ml`, `cup`/`cups`, `tbsp`, `tsp`, `pinch`, `large`, or `""` for counted things. The app can show cups and spoons as metric, and grams as ounces, for the user.
- `quantity` can be `null` for "to taste" (put the words in `unit`).
- A recipe that breaks a rule is skipped by the app (the others still work). Settings > Check for updates says how many were skipped.

## Pictures

1. Make the picture **landscape, about 1200 px wide, WebP (or JPG/PNG), under 300 KB** (the app refuses files over 2 MB).
2. Name it with lowercase letters, digits, `_` or `-`, for example `chicken_tikka.webp`, and put it in `images/`.
3. In the recipe write `"image": "images/chicken_tikka.webp"`.
4. Raise `contentVersion`, commit and push. Push the picture and the JSON in the same commit.

Recipes without an `image` show a neutral placeholder. A picture that is missing on the server is simply skipped and fetched again at the next check, so a late upload is fine.

Only use pictures you have the right to publish.

## Asking users to update the app

The `appUpdate` block at the top of `recipes.json` is **off** (`"enabled": false`). To use it:

```json
"appUpdate": {
  "enabled": true,
  "latestVersionCode": 4,
  "latestVersionName": "0.4.0",
  "minVersionCode": 3,
  "force": false,
  "title": "Update available",
  "message": "A new version is ready. Please update.",
  "url": "https://your-download-page"
}
```

| Field | Meaning |
| --- | --- |
| `enabled` | `false` hides everything. |
| `latestVersionCode` | The newest build number (`versionCode` in the app). Users on a lower build see the notice. |
| `minVersionCode` | Optional. Users **below** this build cannot dismiss the notice (a required update). |
| `force` | `true` makes the notice required for everyone below `latestVersionCode`; `false` lets them tap "Later". |
| `url` | Where "Update now" goes. Must start with `https://`. |
| `title`, `message`, `latestVersionName` | Text shown on the screen (optional). |

- A **required** update covers the whole app and Back leaves the app. It works offline too: the app remembers the last notice it downloaded.
- "Later" is remembered per version: the notice returns when you publish a newer `latestVersionCode`.
- To turn it off, set `"enabled": false` (or delete the block). Users who already updated never see it again.
- Build numbers: 0.2.0 is 2, 0.3.0 is 3. Check `versionCode` in `app/build.gradle.kts`.
