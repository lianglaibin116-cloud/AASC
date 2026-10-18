# Third-Party Notices

The MIT license in [LICENSE](LICENSE) applies to original AASC code and documentation. It does not replace licenses for third-party components. Check the notices for the exact versions resolved by the manifests and lockfile before redistributing a build.

## Optional scanner programs

These scanner CLIs are installed and run separately by the user. They are not bundled with AASC, and this repository grants no rights to redistribute them. Their own licenses apply when you use or distribute each program.

| Program | Upstream license | License reference |
| --- | --- | --- |
| Cisco Skill Scanner | Apache-2.0 | [LICENSE](https://github.com/cisco-ai-defense/skill-scanner/blob/main/LICENSE) |
| Semgrep CLI | LGPL-2.1; upstream may include file-level notices | [COPYRIGHT](https://github.com/semgrep/semgrep/blob/develop/COPYRIGHT) |
| Gitleaks | MIT | [LICENSE](https://github.com/gitleaks/gitleaks/blob/master/LICENSE) |
| OSV-Scanner | Apache-2.0 | [LICENSE](https://github.com/google/osv-scanner/blob/main/LICENSE) |

AASC invokes the local CLI; it does not download scanner programs. Install them only from official upstream releases and review the license for the version you use.

## Application dependencies

The desktop app includes or links dependencies resolved through `frontend/package-lock.json`, `frontend/src-tauri/Cargo.toml`, and `backend/requirements*.txt`. Direct dependencies include Vue (MIT), FastAPI (MIT), Uvicorn (BSD-3-Clause), Pydantic (MIT), PyInstaller (GPL-2.0-or-later with the Bootloader exception; some runtime-hook files are Apache-2.0), Tauri (MIT or Apache-2.0), `tauri-plugin-shell` (MIT or Apache-2.0), Serde (MIT or Apache-2.0), and UUID (MIT or Apache-2.0). Transitive packages can have other terms.

This project does not modify PyInstaller. Review the upstream exception and all bundled dependency notices before redistribution. This summary is not legal advice or a complete bill of materials; verify the exact versions and notices for your build.

Upstream references: [Vue](https://github.com/vuejs/core/blob/main/LICENSE), [FastAPI](https://github.com/fastapi/fastapi/blob/master/LICENSE), [Uvicorn](https://github.com/encode/uvicorn/blob/master/LICENSE.md), [Pydantic](https://github.com/pydantic/pydantic/blob/main/LICENSE), [PyInstaller](https://github.com/pyinstaller/pyinstaller/blob/develop/COPYING.txt), [Tauri](https://github.com/tauri-apps/tauri), [Tauri shell plugin](https://github.com/tauri-apps/plugins-workspace), [Serde](https://github.com/serde-rs/serde/blob/master/LICENSE-MIT), [UUID](https://github.com/uuid-rs/uuid).
