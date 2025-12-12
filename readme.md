[minimal-settings.json](./minimal-settings.json) includes the default config recommended in the [Zed docs](https://zed.dev/docs/languages/html#lsp-formatting) to use `vscode-html-language-server` instead of Prettier for formatting HTML. When using this, Zed doesn't format HTML code.

## Steps to reproduce issue
1. Backup Zed settings, then paste contents of [minimal-settings.json](./minimal-settings.json) into `settings.json`.
2. You may need to restart Zed. I had issues reproducing this without restarting (caused by the language server already running in the background?)
3. Open [test.html](./test.html)
4. Go to `Language Servers`(Ctrl+Alt+L) in the bottom toolbar and ensure `vscode-html-language-server` is running. If it's not, saving the settings file seems to force it to load.
5. Save the test file (to trigger formatting) or run `editor:format`(Ctrl+Shift+I) in the command palette. This will not format the file.

## Solution
1. Use the contents of [fixed-settings.json](./fixed-settings.json) in your settings.
2. Save the test file, which now gets formatted properly.

## Notes
- The fixed config declares the language server explicitly, excludes prettier just in case, and the `"..."` is added to ensure other HTML language servers (like emmet, tailwind) also get attached. You can see for yourself by removing the `"..."` and trying to use emmet snippets. [Zed docs regarding this](https://zed.dev/docs/configuring-languages#choosing-language-servers)
- To further verify the solution, in the command palette, you can run `dev: open language server logs` and select `vscode-html-language-server` in the dropdown. When attempting to format an html file, it should send a request with `"method":"textDocument/formatting"`. This doesn't happen when using the first config but does with the second one.

## Extra fix for LSP error regarding valid css properties
`test.html` includes an empty `style` attribute on purpose. View the LSP logs using the steps mentioned above, interact with the html file, and search for `validProperties` in the logs, which shows this error:
```
Error while validating file:///home/sujal/.config/zed/zed-html-buggy-config/test.html: Cannot read properties of null (reading 'validProperties')
TypeError: Cannot read properties of null (reading 'validProperties')
```
This seems to be thrown when html files have a style tag or attribute.
Not sure if it would cause any issues, but a common workaround seems to be to set `validProperties` to an empty object. Shown in [./fix-css-lsp.json](./fix-css-lsp.json). Apply those settings, restart Zed and view the LSP logs to check that the error no longer shows. You may have to save the settings file to force the LSP to load. Based on this [neovim example config](https://neovim.discourse.group/t/help-with-diagnostics-for-html-files/2682/2).
