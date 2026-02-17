# Stash - Personal Read-It-Later App

## Project Overview
Self-hosted read-it-later application with Chrome extension, iOS Shortcut, and web interface. Built with vanilla JavaScript, Supabase backend, and deployed on Vercel.

## Tech Stack
- **Frontend**: Vanilla JavaScript, HTML, CSS (no framework)
- **Backend**: Supabase (PostgreSQL + Edge Functions)
- **Deployment**: Vercel (web app), Chrome Web Store (extension)
- **Services**: Resend (email digests), Edge TTS (text-to-speech)

## Key Configuration Files
- `extension/config.js` - Chrome extension config
- `web/config.js` - Web app config (same credentials)
- `supabase/functions/save-page/index.ts` - Article extraction Edge Function
- `tts/tts.py` - Text-to-speech generation script

## Credentials & Setup
**Supabase Project:**
- URL: https://vtkxlivxfepxzqibvuto.supabase.co
- Project Ref: `vtkxlivxfepxzqibvuto`
- User ID: `80f5ed87-4e00-4936-9287-18b47fbfe1e4`
- Anon Key: (in config files)

**Deployments:**
- Web App: https://my-stash-rust.vercel.app
- GitHub: https://github.com/joyofcode/MyStash

## Architecture Decisions

### Single-User Mode
- RLS policies set to `true` (allow all operations)
- User ID hardcoded in configs
- No authentication required in UI
- Suitable for personal use only

### iOS Shortcut Authentication
- Uses Edge Function `/functions/v1/save-page`
- Requires only `Authorization: Bearer <anon_key>` header
- Uses `Shortcut Input` variable directly (not Get URLs action)
- 3 actions total: Receive → API Call → Notification

### Chrome Extension
- Does local article extraction using Readability.js
- Direct database inserts via Supabase REST API
- Extracts content in browser, then saves with full article data

### Tag System
- Modal-based UI with search/filter
- 12-color auto-assigned palette
- Multi-select with visual feedback
- Popular tags section

## Key Files Modified

### Database Schema
- `supabase/schema.sql` - Main schema with RLS policies
- Tables: saves, tags, folders, save_tags, user_preferences
- Full-text search using PostgreSQL tsvector

### Web App
- `web/app.js` - Main application logic
  - `openReadingPane` is async (critical for tag loading)
  - `loadSaves` filters by user_id
  - Comprehensive tag modal system
- `web/index.html` - Auth screen hidden by default
- `web/styles.css` - Tag system styles (~250 lines)

### Edge Functions
- `save-page` - Article extraction and saving
  - CORS allows: authorization, apikey, content-type
  - Uses SERVICE_ROLE_KEY internally
  - Accepts: user_id, url, source (and optional prefetched data)

### TTS
- On-demand generation with `--article-id` parameter
- Uses Microsoft Edge TTS (free)
- Uploads to Supabase Storage bucket 'audio'

## Common Tasks

### Deploy Edge Function
```bash
cd C:/Users/brijeshr/stash
supabase functions deploy save-page
```

### Test Edge Function
```bash
curl -X POST "https://vtkxlivxfepxzqibvuto.supabase.co/functions/v1/save-page" \
  -H "Authorization: Bearer <anon_key>" \
  -H "Content-Type: application/json" \
  -d '{"user_id":"80f5ed87-4e00-4936-9287-18b47fbfe1e4","url":"https://example.com","source":"test"}'
```

### Generate Audio for Article
```bash
cd C:/Users/brijeshr/stash/tts
python tts.py --article-id <article-id>
```

### Deploy Web App
Automatic via Vercel GitHub integration - just push to main branch.

## Known Issues & Solutions

### iOS Shortcut Common Problems
1. **"url and user_id required"** - Variable not selected correctly
   - Solution: Tap field and select "Shortcut Input" variable (blue pill)
2. **401 Invalid JWT** - Missing Authorization header
   - Solution: Add `Authorization: Bearer <token>` header
3. **Blank URLs variable** - Get URLs action issues
   - Solution: Use "Shortcut Input" directly, skip Get URLs step

### Web App Issues
1. **Articles not loading** - Missing user_id filter
   - Fixed: `.eq('user_id', this.user.id)` in loadSaves
2. **Tag modal errors** - openReadingPane not async
   - Fixed: Made function async to support await calls

### RLS Policy Issues
- Original policies checked `auth.uid()` but using anon key
- Solution: Changed all policies to `with check (true)` / `using (true)`

## Development Notes

### Branding
- Changed "Made by Kevin Roose" to "Made by Brijesh Warrier"
- Favicon: Purple "S" logo (SVG)

### Preferences
- User prefers concise code without unnecessary comments
- No emojis in code unless explicitly requested
- Focus on functionality over documentation

## Future Improvements
- [ ] Set up digest email cron job (currently deferred)
- [ ] Consider multi-user authentication if needed
- [ ] Add more tag color options
- [ ] Implement folder organization UI
- [ ] Add search functionality to main view

## Useful Commands
```bash
# Start local Supabase (if needed)
supabase start

# View function logs
supabase functions logs save-page

# Push database changes
supabase db push

# Link project (if needed)
supabase link --project-ref vtkxlivxfepxzqibvuto
```

## Notes
- Extension works in Chrome - saves with full content extraction
- iOS Shortcut works - saves URL which can be read in web app
- Web app deployed and accessible as PWA on mobile
- All components tested and functional
