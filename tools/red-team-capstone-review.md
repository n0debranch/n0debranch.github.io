# Red Team Capstone editorial and technical review

Reviewed on 7 November 2026. The user approved publication as a partial lab write-up.

## Changes

- Rewrote the prose for clarity and consistent chronology; retained all 86 original figures in order.
- Made partial completion explicit in the title, introduction, conclusion, and evidence table.
- Distinguished the simulated SWIFT application from real financial messaging.
- Corrected host-discovery scope, VPN configuration/certificate terminology, SMB signing versus encryption, UAC versus administrative membership, and writable service binaries versus unquoted service paths.
- Corrected the PowerUp platform description, AMSI claims, Defender claims, and the interpretation of service restart failure.
- Reported the LDAP and forest-resolution errors visible in the SharpHound screenshot rather than claiming complete collection.
- Identified the unexplained CORP Domain Admin privilege as a missing escalation step, and another participant's involvement as a hypothesis.
- Corrected account SID versus domain SID, RID terminology, Enterprise Admins scope, and domain accounts versus local accounts.
- Replaced the unsupported PsExec description with the PowerShell remoting method visible in the ROOTDC screenshot.
- Repaired wrapped command transcription and a missing IEX character/closing parenthesis; used an explicit placeholder for the changing Kali tunnel address. The original obfuscated AMSI excerpt is retained separately as unverified historical material.
- Replaced the malformed Install-ServiceBinary link and indirect references with 17 direct room/project/vendor references. Internal lab URLs are code, not public links.
- Removed the student number and registration-password line from the public prose.

## Verification

- Jekyll build passed with the existing Chirpy theme.
- All 86 image references and order matched the prior conversion.
- Built HTML has no missing internal links/assets, duplicate IDs, or missing image alt text.
- Ten corrected PowerShell blocks parsed successfully without execution. This does not validate tool arguments or live lab behaviour.
- Fifteen external reference URLs returned HTTP 200. TryHackMe returned 429 and Swift returned 403 to the automated checker; both exact pages were separately verified through web retrieval.
- Desktop (1440px) and mobile (390px) browser review passed: no document-width overflow or out-of-viewport images. The evidence table wraps on desktop and scrolls on narrow screens; long commands scroll within their blocks.
- Local HTMLProofer remains blocked by an incompatible Windows libcurl dependency. The repository's Linux GitHub Actions build runs HTMLProofer before deployment.

## Limits

No lab commands were executed during this review. No fresh end-to-end room attempt was performed. The corporate privilege transition, complete collector run, cleanup, and final SWIFT stage remain unverified and are explicitly identified in the article. Screenshots retain their original historical content.
