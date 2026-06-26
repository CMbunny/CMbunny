# NVS (Non-Volatile Storage)
NVS is the "hard drive" of your ESP32. Since microcontrollers lose all RAM data when powered off, NVS provides a way to save small amounts of data (like Wi-Fi credentials, user settings, or authentication tokens) that survive reboots.

**1. The Core Architecture** <br>
***Namespaces*** <br>
Think of NVS like a Windows registry or a folder system. A Namespace is a container (folder) that holds a collection of key-value pairs.
- **Why use them?**: To prevent collisions. You might have a system_config namespace and an auth_data namespace. Even if both have a key named "version", they won't overwrite each other.<br>

***Blob vs. String*** <br>
- **String:** Used for text. It is null-terminated and limited to the size defined in your storage.
- **Blob (Binary Large Object):** Used for structs, arrays, or raw binary data. It stores exact byte-for-byte memory copies. This is the most efficient way to store a hmi_user_t struct, as you save the entire memory block in one operation.<br>

***Wear Leveling*** <br>
Flash memory chips have a limited number of "write cycles" before they wear out. If you constantly write to the same physical address, that sector will die quickly.
- **How NVS solves this:** It uses a "Log-structured" system. When you update a key, NVS doesn't overwrite the old data; it writes the new data to a fresh, empty page and marks the old one as "invalid." The internal Wear Leveling algorithm handles this by rotating which physical sector is used, spreading the wear evenly across the entire partition.<br>

**Question:** What happens when the NVS partition is full?<br>
**Answer:** When the partition runs out of erased pages: <br>
**1.)Garbage Collection:** NVS triggers an internal "compaction" process. It takes all the valid (current) data, moves it to a fresh page, and erases the old pages that contained "invalid" (stale) data.
**2.)Failure:** If even after compaction there is no space, operations will return ESP_ERR_NVS_NOT_ENOUGH_SPACE. <br>

**2. Implementation: Basic to Advanced** <br>
***Initialization and Setup*** <br>
Before using NVS, it must be initialized. In your partition table, you must define an `nvs` partition type.
```
#include "nvs_flash.h"

void init_nvs() {
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        // Partition was truncated or full, erase and retry
        ESP_ERROR_CHECK(nvs_flash_erase());
        ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);
}
```
***Storing Data (Blob Example)*** <br>
Storing a struct as a blob is the professional way to handle configuration data.

```
void save_user_data(hmi_user_t *user) {
    nvs_handle_t my_handle;
    nvs_open("hmi_auth", NVS_READWRITE, &my_handle);
    
    // Store struct as a blob
    nvs_set_blob(my_handle, "admin_user", user, sizeof(hmi_user_t));
    nvs_commit(my_handle); // Mandatory to push to flash
    nvs_close(my_handle);
}
```
