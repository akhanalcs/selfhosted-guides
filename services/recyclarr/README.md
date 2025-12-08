# Recyclarr
This tool automatically syncs TRaSH Guides (Media Naming, File Sizes, Custom Formats, Quality Profiles) to your Radarr and Sonarr instances. 
It ensures your library quality standards are always up to date without manual tweaking.

This guide focuses on setting up Recyclarr for a single Radarr instance (combining 1080p and 4K profiles), which is sufficient for majority of users.

- Some people run 3 instances of Radarr for different quality profiles (e.g., 1080p, 4K, Anime). 
- But I like to keep it simple and just run 1 instance of Radarr with 1080p and 4K profiles.
- I don't watch Anime, so I don't need a separate Radarr instance for that. If I were to watch Anime movies, I would just add another instance of Radarr and use Recyclarr to sync the Anime profiles there.

Check out their [docs](https://recyclarr.dev/guide/features) for more info.

## Steps
1. Make sure you have Radarr running. If not, follow the [Radarr setup guide](../radarr/README.md).
2. Add recyclarr to the list in main `.env` file
   ```env
   COMPOSE_PROFILES="required,...,recyclarr"
   ```
3. Give ownership of recyclarr data folder to your user
   ```bash
   $ sudo chown -R $(id -u):$(id -g) -R /opt/docker/data/recyclarr
   ```
4. Create `recyclarr/secrets.yml` file and set the following:
    - `radarr_main_base_url`: `http://radarr:7878`
    - `radarr_main_api_key`: Get it from Radarr > Settings > General > Security > API Key
5. Create an empty `.yml` file: `recyclarr/recyclarr.yml` that we will populate in the next steps.
   For the complete file, check out my [recyclarr.yml](recyclarr.yml) file.

   Populate just the update options for now:
   ```yml
   radarr:
     radarr_main:
       delete_old_custom_formats: true
       replace_existing_custom_formats: true
   ```
6. Populate media naming in `recyclarr.yml`. 
   What to populate here is found by running the CLI command below:
   ```bash
   $ docker compose run --rm recyclarr list naming radarr
   ```
   This gives the following output:
   ```
   Media Naming Formats (Preview)
   
   Movie Folder Format
   ┌───────────────┬───────────────────────────────────────────────────────┐
   │ Key           │ Format                                                │
   ├───────────────┼───────────────────────────────────────────────────────┤
   │ plex-tmdb     │ {Movie CleanTitle} ({Release Year}) {tmdb-{TmdbId}}   │
   │ (...other formats removed for brevity)                                │
   └───────────────┴───────────────────────────────────────────────────────┘
   
   Standard Movie Format
   ┌───────────────┬────────────────────────────────────────────────────────┐
   │ Key           │ Format                                                 │
   ├───────────────┼────────────────────────────────────────────────────────┤
   │ plex-tmdb     │ {Movie CleanTitle} {(Release Year)} {tmdb-{TmdbId}} -  │
   │               │ {edition-{Edition Tags}} {[MediaInfo 3D]}{[Custom      │
   │               │ Formats]}{[Quality Full]}{[Mediainfo AudioCodec}{      │
   │               │ Mediainfo AudioChannels]}{[MediaInfo                   │
   │               │ VideoDynamicRangeType]}{[Mediainfo                     │
   │               │ VideoCodec]}{-Release Group}                           │
   │ (...other formats removed for brevity)                                 │
   └───────────────┴────────────────────────────────────────────────────────┘
   ```
   Based on this output, populate media naming in `recyclarr.yml` like below. I'm using Plex, so I'm choosing `plex-tmdb` for both folder and movie naming. For your setup, choose whatever you like from the available options.
   ```yml
    # Seen in Radarr > Settings > Media Management > Movie Naming
    media_naming:
      folder: plex-tmdb
      movie:
        rename: true
        standard: plex-tmdb
   ```
7. With `recyclarr/compose.yaml`, `recyclaar/secrets.yml` and `recyclaar/recyclarr.yml` in place, start Recyclaar in Cron mode:
   ```bash
   $ docker compose up -d recyclarr
   ```
8. Now let's populate the `recyclarr.yml` file with [TRaSH quality profiles for 1080 and 4K](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#trash-quality-profiles) using `include` templates.
   - Run this command to see the available templates that match TRaSH Guide quality profiles.
     ```bash
     $ docker exec recyclarr recyclarr config list templates
     ```
   
     This will be the output. I've removed the language specific and SQP templates for brevity.
     ```
     ┌────────────────────────────┬────────────────────────────────────────┐
     │ Sonarr                     │ Radarr                                 │
     ├────────────────────────────┼────────────────────────────────────────┤
     │ web-1080p-v4               │ hd-bluray-web    👈 Note this name     │
     │ web-2160p-v4               │ uhd-bluray-web   👈 Note this name     │
     │ anime-sonarr-v4            │ remux-web-1080p  👈 Note this name     │
     │                            │ remux-web-2160p  👈 Note this name     │
     │(...lang specific templates)│ anime-radarr                           │
     │                            │                                        │
     │                            │ (...lang specific templates)           │
     │                            │ (...sqp-templates)                     │
     └────────────────────────────┴────────────────────────────────────────┘
     ```
     Note the names of the templates that you want as you'll need them to find the corresponding `include` templates in the next step.
     
     If I only needed one of those templates, I would just generate config file in a very straightforward way using this command
     ```bash
     $ docker exec recyclarr recyclarr config create --template hd-bluray-web
     ```
     But since I need all of them in my 1 Radarr instance, I need to find the corresponding `include` templates for each of them.
   - Run this command to see the IDs of all the include templates
     ```bash
     $ docker exec recyclarr recyclarr config list templates --includes
     ```
    
     This will be the output. I've only included the Radarr column and removed the language specific and SQP Ids for brevity.
     ```
     ┌──────────────────────────────────────────────────────────────┐
     │ Radarr                                                       │
     ├──────────────────────────────────────────────────────────────┤
     │ radarr-custom-formats-anime                                  │
     │ radarr-custom-formats-hd-bluray-web                          │
     │ radarr-custom-formats-remux-web-1080p                        │
     │ radarr-custom-formats-remux-web-2160p                        │
     │ radarr-custom-formats-uhd-bluray-web                         │
     │ (...sqp custom-formats)                                      │
     │ radarr-quality-definition-movie                              │
     │ radarr-quality-definition-anime                              │
     │ (...sqp quality-definition streaming)                        │
     │ (...sqp quality-definition uhd)                              │
     │ radarr-quality-profile-anime                                 │
     │ radarr-quality-profile-hd-bluray-web                         │
     │ radarr-quality-profile-remux-web-1080p                       │
     │ radarr-quality-profile-remux-web-2160p                       │
     │ radarr-quality-profile-uhd-bluray-web                        │
     │ (...sqp quality-profiles)                                    │
     └──────────────────────────────────────────────────────────────┘
     ```

     Here it is visualized in a tree structure to see it better. 

     We're going to "include" the `custom-formats` and `quality-profiles` of the templates we noted earlier in our `recyclarr.yml` file to do rest of our setup.
     ```
     radarr
     ├── includes
     │   ├── custom-formats
     │   │   ├── sqp
     │   │   │   └── ...
     │   │   ├── radarr-custom-formats-anime.yml
     │   │   ├── radarr-custom-formats-hd-bluray-web.yml    👈 Use this in include for "HD Bluray + WEB (1080p)"
     │   │   ├── radarr-custom-formats-remux-web-1080p.yml  👈 Use this in include for "Remux + WEB 1080p"
     │   │   ├── radarr-custom-formats-remux-web-2160p.yml  👈 Use this in include for "Remux + WEB 2160p"
     │   │   └── radarr-custom-formats-uhd-bluray-web.yml   👈 Use this in include for "UHD Bluray + WEB (4K)"
     │   │
     │   ├── quality-definitions
     │   │   ├── sqp
     │   │   │   └── ...
     │   │   ├── radarr-quality-definition-anime.yml
     │   │   └── radarr-quality-definition-movie.yml    👈 Use this in include
     │   │
     │   └── quality-profiles
     │       ├── sqp
     │       │   └── ...
     │       ├── radarr-quality-profile-anime.yml
     │       ├── radarr-quality-profile-hd-bluray-web.yml      👈 Use this in include for "HD Bluray + WEB (1080p)"
     │       ├── radarr-quality-profile-remux-web-1080p.yml    👈 Use this in include for "Remux + WEB 1080p"
     │       ├── radarr-quality-profile-remux-web-2160p.yml    👈 Use this in include for "Remux + WEB 2160p"
     │       └── radarr-quality-profile-uhd-bluray-web.yml     👈 Use this in include for "UHD Bluray + WEB (4K)"
     ```
   - Now based on this information, we can populate the `include`s in `recyclarr.yml` file quite easily.
     ```yml
     include:
       # Seen in Radarr > Settings > Quality
       - template: radarr-quality-definition-movie
        
       # HD Bluray + WEB (1080p)
       # Seen in Radarr > Settings > Profiles
       - template: radarr-quality-profile-hd-bluray-web
       # Seen in Radarr > Settings > Custom Formats
       - template: radarr-custom-formats-hd-bluray-web
        
       # UHD Bluray + WEB (4K)
       - template: radarr-quality-profile-uhd-bluray-web
       - template: radarr-custom-formats-uhd-bluray-web
        
       # Remux + WEB 1080p
       - template: radarr-quality-profile-remux-web-1080p
       - template: radarr-custom-formats-remux-web-1080p
        
       # Remux + WEB 2160p
       - template: radarr-quality-profile-remux-web-2160p
       - template: radarr-custom-formats-remux-web-2160p
     ```
9. That's it! You have all the recommended TRaSH configs in place!
   - If you want to customize it further, check out
     - [Quick Setup Templates](https://recyclarr.dev/guide/guide-configs)
     - [Configuration Examples](https://recyclarr.dev/reference/config-examples) - It shows different scenarios and also shows how to combine multiple configs into a single file for a single instance.
   - For 99% of users, this is enough.
10. If you created these files locally on your computer, upload them to your server now.
    - If you decide to use my folder, copy the `recyclarr` folder except the `screenshots` folder and the `README.md` file.
11. It syncs on a schedule, but you can force it to sync now
    ```bash
    $ docker exec recyclarr recyclarr sync
    ```
    You should see something like this:
    ```
    ===========================================
    Processing Radarr Server: [radarr_main]
    ===========================================
    
    [INF] Created 41 New Custom Formats
    [INF] Total of 41 custom formats were synced
    [INF] Created 4 Profiles: ["HD Bluray + WEB","UHD Bluray + WEB","Remux + WEB 1080p","Remux + WEB 2160p"]
    [INF] A total of 4 profiles were synced. 4 contain quality changes and 4 contain updated scores
    [INF] Total of 14 sizes were synced for quality definition movie
    [INF] Media naming has been updated
    [INF] Completed at 12/07/2025 00:03:44
    ```
12. Check out Radarr now. You should see all the new Quality Definitions, Quality Profiles, Custom Formats and Media Naming applied by Recyclarr! 🎉
    - Movie Naming (Settings > Media Management > Movie Naming)
      
      <img src="screenshots/rad-movie-naming.png" alt="image" width="2950"/>
    - Profiles (Settings > Profiles)
      
      <img src="screenshots/rad-quality-profiles.png" alt="image" width="3614"/>
    - Quality Definitions (Settings > Quality)
      
      <img src="screenshots/rad-quality-def.png" alt="image" width="3326"/>
    - Custom Formats (Settings > Custom Formats)
    
      <img src="screenshots/rad-custom-formats.png" alt="image" width="3642"/>
13. You're done with Recyclarr setup! It will run every day to sync any updates from TRaSH Guides automatically.
