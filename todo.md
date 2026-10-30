
* look add claude code Skill injection and opencode commands. especially for handover between skills/agents
* orchestrator session: nimmt anweisung von user entgegeben und spawn subagent für quellcode exploration, recherche, architectur, imeplementierung, review, etc. Idee ist, dass der orchestrater selbst nicht mit dem projekt interagiert, aber dafür ein teures llm ist. insb. soll es dann auch feedback cycles geben. z.B. wenn implementierung merkt, dass die architektur nicht funktioniert, dann soll eine neue recherche gestartet werden, und dann ggf. eine architectur überlegt werden. 
* mattpocock adaptieren, insb.  https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md
* serverops skill: kümmert sich um deployment. insb schaut es sich merge-notes an und startet server neu mit neuen features. und schaut ins log nach fehlern/bugs, und schaut nach einem merge, ob das log die erwarteten neuen zeilen hat. Idee ist, dass der server local oder per ssh erreichbar ist, und dass die session/der agent messages von anderen session/agenten/oder direkt vom user bekommt, die dann deployment anfragen. 
* alle meine projekte durchsuchen und skills hier consolidieren
* multi-agent review skill hier herkopieren
 * extra agent: tension between claude.md und implementierung hier oder requirements.
 * falls reviewmd vorhanden: extra agent dafür.
 * extra agent: was sind tangierende architektur entscheidungen? --> recherche, wie andere projekte ähnliche probleme lösen.
* git skill/agent: read-only git access (e.g. diff of changes). Use it in the implement skill so the reviewer sees the actual diff instead of relying on the implementer's list of changed files.
* skill, der agentsmd/claudemd reviewt. anschauen wie eine gute agentsmd aussieht, z.b. https://github.com/bjuergens/comibull/blob/main/CLAUDE.md und dann prinzipien in den skill festhalten.
  * lean
  * tension zu review? code? anderen documenten? systemprompt/harness?
  * Zu detailliert?
  * 
  * inhalt
    * 1-2 satze was die anwendung macht
    * grober techstack
    * grobes deployment
    * design prinzipien
    * Do und Donts
    * Tooling 
    * Tests/validation
    * optional: präfix für AI-kommentare und ai-dokumentation
    * optional: wie soll geloggt werden?
    * optional: erwartete blindspots (z.B. keine deployment logs. oder repo files die ignroiert werden kontext pollution)
    * falls agent committen darf, dann anweisung für commitnachricht, und wann commited werden soll.
    * enthält root agents.md details für verschiebende Subssystem (z.B. backend UND frontend?) --> splitten
  * falscher Inhalt:
    *  reviewhinweise. Einige reviewhinweise verschlechtern die qualität von implementern. z.B. wenn man ihm sagt, dass er nur abstractionen hinzufügen soll, wenn es sinnvoll ist, dann macht er fast niemals neue abstractionen. Wenn man ihm sagt, dass er schauen soll, wo abstractionen sinnvoll sind, dann baut er überall abstractionen ein. Das LLM plant nicht vorraus und kann vorab nicht sagen ob eine Abstraktion sinnvoll ist oder nicht. --> am besten man lässt ihn erstmal irgendwas machen, und dann schaut man im review ob die abstraktion gut war
    * Details, wo sich z.B. der linter drum kümmert
    * Details, die nur in spezifischen situationen relevant sind, und wo das llm selbst drauf kommen kann
    * Details, die nur für einzelne untermodule relevant sind --> ordner von untermodul auslagern
    * mehr?
   




 
