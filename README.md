package main

import ("context,","fmt",log","os"",time")

    "github.com/syncthing/syncthing/lib/config"
    "github.com/syncthing/syncthing/lib/daemon"
    "github.com/syncthing/syncthing/lib/options"
    "github.com/syncthing/syncthing/lib/path"
    "github.com/syncthing/syncthing/lib/sqldb"
    "github.com/syncthing/syncthing/lib/util"
)

func main() {
    // Create a new configuration.
    cfg := config.New()
    cfg.SetDefaults()

    // Set the Syncthing ID.
    cfg.Global.ID = "YOUR_SYNCTHING_ID"

    // Set the Syncthing API key.
    cfg.Global.APIKey = "YOUR_SYNCTHING_API_KEY"

    // Add a new folder to Syncthing.
    cfg.Folders.Add(config.Folder{
        ID:        "myfolder",
        Path:      "/path/to/your/folder", // Replace with the actual path
        FolderType: config.FolderTypeLocal,
        DeviceIDs: []string{
            "DEVICE_ID_1", // Replace with the device IDs to sync with
            "DEVICE_ID_2", // Replace with the device IDs to sync with
        },
    })

    // Create a new database connection.
    db, err := sqldb.Open(cfg.Global.Database.Path, cfg.Global.Database.Type)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Create a new options instance.
    opts := options.New(cfg, db, path.NewFs())

    // Start the Syncthing daemon.
    ctx := context.Background()
    d := daemon.New(ctx, opts)
    if err := d.Start(); err != nil {
        log.Fatal(err)
    }

    // Wait for the daemon to start.
    time.Sleep(5 * time.Second)

    // Stop the daemon.
    d.Stop()

    // Print the status of the daemon.
    fmt.Println("Syncthing daemon status:", d.Status())

    // Check for errors.
    if err := d.Err(); err != nil {
        log.Fatal(err)
    }
}
