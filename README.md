# Jump back in
This is a simple browser start/new tab page used with [ActivityWatch](https://activitywatch.net/), an open-source time tracking app.
Jump back in queries ActivityWatch (specifically, the things tracked from your browser) and suggests you... jump back in. (into the things you spent the most time on yesterday and today)

## Using

Jump back in is available at my [public instance](https://martcho.uk/app/jumpbackin), although you need to allow my domain as a CORS origin in your local ActivityWatch configuration, as such:
- `cd ~/.config/activitywatch/aw-server`
- edit the `aw-server.toml` file to have `"https://martcho.uk"` as one of the `cors_origins`
- if this is your first time editing this config file, you might need to uncomment (remove the `#`) the 5 lines for `[server]`

If this is too much of a security risk for you, you can clone the `build` branch of this repo and host the files yourself, or clone the `main` branch and host a local webserver with `npm install && npm run dev`.
You might need to add different `cors_origins` in your config then. Due to how CORS works, it's not possible, unfortunately, to use this from a local index.html.

![Screenshot 2024-04-10 at 17-57-03 Vite Svelte](https://github.com/janmartchouk/jumpbackin/assets/19735475/c5f38535-20ce-4b78-aff3-4a85476efd39)
![Screenshot 2024-04-10 at 17-57-12 Vite Svelte](https://github.com/janmartchouk/jumpbackin/assets/19735475/55db2781-c383-41ec-99e0-55f1e52328c1)

