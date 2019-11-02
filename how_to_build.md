# Documentation Site Build and Deploy Instructions

1. Install Choclately
   1. Install via Powershell `Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`
   1. Detailed instructions found at - <https://chocolatey.org/install>
1. Install DocFx via Chocolately
   1. In Powershell run `choco install docfx`
1. Clone RogueSharp
   1. `git clone git@github.com:FaronBracy/RogueSharp.git`
1. Clone RogueSharp.Documentation
   1. `git clone git@github.com:FaronBracy/RogueSharp.Documentation.git`
   1. Ensure that the root directories for both RogueSharp and RogueSharp.Documentation are at the same level.
1. Build the API Documentation
   1. Navigate to the RogueSharp.Documentation root directory
   1. Run `docfx metadata`
1. Build the DocFx site
   1. Run `docfx build`
1. Serve the site locally to preview
   1. Run `docfx serve .\_site\`
   1. Browse to `http://localhost:8080/` to see the site
1. Publish the DocFx site
   1. Copy everything from the `RogueSharp.Documentation\_site` folder in RogueSharp.Documentation
   2. Paste the contents into the `RogueSharp\Docs` folder
   3. Push to Github
