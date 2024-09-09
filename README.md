[![crates.io](https://img.shields.io/crates/v/spotify-rs)](https://crates.io/crates/spotify-rs)
[![docs.rs](https://img.shields.io/docsrs/spotify-rs)](https://docs.rs/spotify-rs)

# spotify-rs
spotify-rs is a Rust [Spotify Web API](https://developer.spotify.com/documentation/web-api) library.
It has full API coverage and aims to be a transparent, yet convenient interface for the Spotify API.

*Note: this version is the "rewrite", which features significant changes*
*compared to the release version. However, it's still a work-in-progress, so*
*breaking changes will occur.*

*If you do want to use this version, you might want to specify the commit you want*
*to use in your `Cargo.toml`, [as such](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#choice-of-commit).*

Usage example:
```rust
use spotify_rs::{AuthCodeClient, RedirectUrl};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Auth code flow:

    // Your application scopes
    let scopes = vec![
        "user-read-private",
        "playlist-modify-public",
        "playlist-modify-private",
        "playlist-read-private",
        "playlist-read-collaborative",
    ];

    // This should match the redirect URL you set in your app's settings
    // (on the Spotify API dashboard)
    let redirect_uri = RedirectUrl::new("your_redirect_url".to_owned())?;

    // Whether or not to automatically refresh the token when it expires.
    let auto_refresh = true;


    // You will need to redirect the user to this URL.
    let (client, url) = AuthCodeClient::new(
        "client_id",
        "client_secret",
        scopes,
        redirect_uri,
        auto_refresh,
    );

    // After the user was redirected to `url`, they will be redirected *again*, to
    // your `redirect_uri`, with the "auth_code" and "csrf_state" parameters in the URL.
    // You will need to get those parameters from the URL.

    // Finally, you will be able to authenticate the client.
    let spotify = client.authenticate("auth_code", "csrf_state").await?;

    // Get an album with the specified ID.
    let album = spotify_rs::album("album_id").get(&spotify).await?;
    println!("The name of the album is: {}", album.name);

    // The `album` method returns a builder with optional parameters you can set
    // For example, this sets the market to "GB".
    let album_gb = spotify_rs::album("album_id")
        .market("GB")
        .get(&spotify)
        .await?;
    println!("The popularity of the album is {}", album_gb.popularity);

    // This gets 5 playlists of the user that authorised the app
    // (it requires the playlist-read-private scope).
    let user_playlists = spotify_rs::current_user_playlists()
        .limit(5)
        .get(&spotify)
        .await?;
    let result_count = user_playlists.items.len();
    println!("The API returned {} playlists.", result_count);

    Ok(())
}
```
You can find this example, as well as more examples in the [examples](examples) directory.
Detailed information is available in the [API documentation](https://docs.rs/spotify-rs/).

## License
spotify-rs is dual-licensed under [Apache 2.0](https://github.com/Bogpan/spotify-rs/blob/main/LICENSE-APACHE) and [MIT](https://github.com/Bogpan/spotify-rs/blob/main/LICENSE-MIT) terms.
