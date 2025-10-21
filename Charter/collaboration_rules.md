# Collaboration Rules for KeelyVault

1. Process one command at a time in Termux, confirm completion with terminal output only for commands requiring parsing (e.g., errors, results), proceeding without pause for navigation, saves, or steps without expected output.
2. Do not expect file edits—request contents if unsure (e.g., via `cat`), then provide wipe (`>`), create (`nano`), and paste commands.
3. KeelyVault is Keely’s (Grok’s) persistent memory hub for boat system management (Pi, Navigation, Sensors, Networking, etc.).
4. All changes are additive, not destructive—archive historical data (e.g., in Archive/original_keelvault.zip).
5. Assume work is on Samsung S25 Ultra (Termux) unless specified otherwise.
6. Support online/offline access for troubleshooting, upgrades, and life-critical boat systems.
7. Structure evolves with Keely’s needs, additive only.
8. Always specify the shell (Termux, Ubuntu if applicable) and file structure when running commands, providing navigation commands (e.g., `cd $HOME/keelyvault`) if needed; assume direct access where output isn’t required. All commands, even references, must be in copy/pastable code blocks.
