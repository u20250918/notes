好的，明白了。你希望设置一个永久的全局代理，这样每次启动Chromium时就不需要手动设置，并且最好是系统全局的。

**方法 1：配置系统全局代理（推荐）**

这种方法会影响整个系统，包括Chromium和其他应用程序。

1. **设置环境变量：**
   编辑你的shell配置文件（例如`~/.bashrc`、`~/.zshrc`或`~/.profile`），添加以下行：

   ```bash
   export http_proxy="socks5://localhost:9909"
   export https_proxy="socks5://localhost:9909"
   export all_proxy="socks5://localhost:9909"
   export no_proxy="localhost,127.0.0.1,::1" # 可选，添加不需要通过代理的地址
   ```

   * `http_proxy`：用于HTTP连接的代理。
   * `https_proxy`：用于HTTPS连接的代理。
   * `all_proxy`：用于所有协议的代理（如果适用）。
   * `no_proxy`：不需要通过代理的地址列表，用逗号分隔。

   如果你使用HTTP代理，将`socks5`替换为`http`，端口号也相应更改。
   保存文件并执行以下命令使更改生效：

   ```bash
   source ~/.bashrc  # 如果你编辑的是.bashrc
   source ~/.zshrc  # 如果你编辑的是.zshrc
   source ~/.profile # 如果你编辑的是.profile
   ```

   或者，你可以注销并重新登录。
2. **测试：**
   打开一个新的终端窗口，运行`env | grep proxy`，确认环境变量已设置。 然后，启动Chromium，它应该会自动使用系统代理设置。

**方法 2：创建 Chromium 启动器（仅影响 Chromium）**

这种方法只影响 Chromium，不会影响其他应用程序。

1. **找到 Chromium 的启动器文件：**
   通常，启动器文件位于** **`/usr/share/applications/` 或** **`~/.local/share/applications/`。 找到名为** **`chromium.desktop` 或类似的 Chromium 启动器文件。
2. **编辑启动器文件：**
   使用文本编辑器打开启动器文件。 你可能需要使用** **`sudo` 来编辑** **`/usr/share/applications/` 中的文件。
   找到** **`Exec=` 开头的行。 例如：

   ```
   Exec=/usr/bin/chromium %U
   ```

   修改这一行，添加代理参数：

   ```
   Exec=/usr/bin/chromium --proxy-server="socks5://localhost:9909" %U
   ```

   如果需要添加** **`no_proxy`，可以尝试以下方法（可能不适用于所有桌面环境）：

   ```
   Exec=env no_proxy="localhost,127.0.0.1,::1" /usr/bin/chromium --proxy-server="socks5://localhost:9909" %U
   ```

   保存文件。
3. **测试：**
   从桌面环境的菜单或启动器中启动 Chromium。 它应该会自动使用你设置的代理。

**完整命令总结：**

* **方法 1（系统全局）：**

  ```bash
  echo 'export http_proxy="socks5://localhost:9909"' >> ~/.bashrc
  echo 'export https_proxy="socks5://localhost:9909"' >> ~/.bashrc
  echo 'export all_proxy="socks5://localhost:9909"' >> ~/.bashrc
  echo 'export no_proxy="localhost,127.0.0.1,::1"' >> ~/.bashrc # 可选
  source ~/.bashrc
  ```

  （请根据你的 shell 和代理类型进行调整）
* **方法 2（Chromium 启动器）：**

  ```bash
  # 找到 chromium.desktop 文件，例如：
  # sudo nano /usr/share/applications/chromium.desktop
  # 然后修改 Exec= 行，添加 --proxy-server 参数
  # Exec=/usr/bin/chromium --proxy-server="socks5://localhost:9909" %U
  ```

  （请根据你的 Chromium 启动器文件路径和内容进行调整）

**重要注意事项：**

* **安全：** 全局代理会将所有网络流量都通过代理服务器路由。 确保你信任代理服务器，并且了解潜在的安全风险。
* **优先级：** 某些应用程序可能会忽略系统环境变量。 在这种情况下，你需要单独配置这些应用程序的代理设置。
* **Geph 必须运行：** 无论你使用哪种方法，Geph 必须始终运行，否则 Chromium 将无法连接到互联网。
* **测试：** 设置完成后，务必测试代理是否正常工作。 访问** **`https://www.google.com` 或** **`https://www.youtube.com`，并使用** **`https://whatismyipaddress.com/` 确认你的 IP 地址已更改。
* **HTTP 代理：** 如果你使用 HTTP 代理，请将** **`socks5` 替换为** **`http`，并相应地更改端口号。

希望这些详细的步骤和完整的命令能够帮助你设置永久的全局代理！ 选择适合你的需求的方法，并仔细按照步骤操作。
