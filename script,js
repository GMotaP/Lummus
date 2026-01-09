(function () {
  document.addEventListener("DOMContentLoaded", () => {
    const locationsContainer = document.getElementById("locations");
    const updateTimeElement = document.getElementById("last-update");
    const scaleRoot = document.getElementById("scale-root");
    const pageError = document.getElementById("page-error");

    if (!locationsContainer || !updateTimeElement || !scaleRoot) {
      console.error("[INCHARGE] Elementos base não encontrados.", {
        hasLocations: !!locationsContainer,
        hasUpdate: !!updateTimeElement,
        hasScale: !!scaleRoot,
      });
      if (pageError) {
        pageError.style.display = "block";
        pageError.textContent = "Erro: elementos base não encontrados (IDs).";
      }
      return;
    }

    const colunas = [
      {
        name: "Locais",
        locations: [
          { label: "Hotel Santory", key: "inc298" },
          { label: "Hotel Colonial", key: "inc334" },
          { label: "Drogaria Americana", key: "inc335" },
        ],
      },
      {
        name: "Locais",
        locations: [
          { label: "Shopping Fabrika Mall DC", key: "pc111" },
          { label: "Posto Caxuxa", key: "inc299" },
          { label: "Posto One", key: "inc332" },
        ],
      },
    ];

    function isOnlineValue(online) {
      return online === 1 || online === true || online === "1";
    }

    function getStatusClass(status, online) {
      const s = String(status || "").trim().toLowerCase();

      if (s === "faulted" || s === "unavailable" || s === "error") return "is-offline";
      if (!isOnlineValue(online)) return "is-offline";

      switch (s) {
        case "available": return "is-available";
        case "preparing": return "is-preparing";
        case "finishing": return "is-finishing";
        case "charging": return "is-charging";
        default: return "is-available";
      }
    }

    function getPaymentLink(key, plug) {
      const upper = String(key || "").toUpperCase();
      return `https://incharge.app/now/${upper}/${plug}`;
    }

    function atualizarHorario() {
      const agora = new Date();
      const hora = agora.toLocaleTimeString("pt-BR", {
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit",
      });
      const data = agora.toLocaleDateString("pt-BR");
      updateTimeElement.textContent = `Última atualização: ${data} às ${hora}`;
    }

    function fitToViewport() {
      scaleRoot.style.transform = "scale(1)";
      const margin = 8;

      const rect = scaleRoot.getBoundingClientRect();
      const availableH = window.innerHeight - margin;
      const availableW = window.innerWidth - margin;

      let scale = Math.min(1, availableH / rect.height, availableW / rect.width);
      if (!Number.isFinite(scale)) scale = 1;
      if (scale < 0.6) scale = 0.6;

      scaleRoot.style.transform = `scale(${scale})`;
    }

    function renderColunas(globalData, hadAnyError) {
      locationsContainer.innerHTML = "";

      if (pageError) {
        pageError.style.display = hadAnyError ? "block" : "none";
        if (hadAnyError) {
          pageError.textContent =
            "Falha ao carregar dados de alguns/ todos os locais (ver Console). Possível CORS ou endpoint inválido.";
        }
      }

      colunas.forEach((coluna) => {
        const col = document.createElement("div");
        col.className = "city-column";

        const h2 = document.createElement("h2");
        h2.textContent = coluna.name;
        col.appendChild(h2);

        coluna.locations.forEach((loc) => {
          const t = document.createElement("h3");
          t.className = "titleCidade";
          t.textContent = loc.label;
          col.appendChild(t);

          const container = document.createElement("div");
          container.className = "containerInfo";

          const chargers = Array.isArray(globalData[loc.key]) ? globalData[loc.key] : [];

          if (chargers.length === 0) {
            const p = document.createElement("p");
            p.className = "loading";
            p.textContent = hadAnyError
              ? "Falha ao carregar (ver console)"
              : "Sem dados no momento";
            container.appendChild(p);
          } else {
            chargers.forEach((ch) => {
              const linkA = document.createElement("a");
              linkA.href = getPaymentLink(loc.key, ch.plug);
              linkA.target = "_blank";
              linkA.rel = "noopener noreferrer";

              const item = document.createElement("div");
              item.className = "chargerInfo " + getStatusClass(ch.status, ch.online);
              item.textContent = `Plug ${ch.plug}`;

              if (!isOnlineValue(ch.online)) item.style.opacity = "0.6";

              linkA.appendChild(item);
              container.appendChild(linkA);
            });
          }

          col.appendChild(container);
        });

        locationsContainer.appendChild(col);
      });
    }

    let isFetching = false;

    async function getAllData() {
      if (isFetching) return;
      isFetching = true;

      let hadAnyError = false;

      try {
        const allKeys = [...new Set(colunas.flatMap((c) => c.locations.map((l) => l.key)))];

        const urls = allKeys.map((key) => ({
          key,
          url: `https://api.incharge.app/api/v2/now/${key}`,
        }));

        const responses = await Promise.all(
          urls.map(async (item) => {
            try {
              const res = await fetch(item.url, { cache: "no-store" });
              if (!res.ok) throw new Error(`HTTP ${res.status}`);

              const data = await res.json();

              const parsed = Array.isArray(data)
                ? data
                : Array.isArray(data?.chargers)
                ? data.chargers
                : [];

              return { key: item.key, data: parsed };
            } catch (err) {
              hadAnyError = true;
              console.error("[INCHARGE] Falha ao buscar", item.key, err);
              return { key: item.key, data: [] };
            }
          })
        );

        const globalData = {};
        responses.forEach((r) => (globalData[r.key] = r.data));

        renderColunas(globalData, hadAnyError);
        atualizarHorario();
        fitToViewport();
      } catch (e) {
        hadAnyError = true;
        console.error("[INCHARGE] Erro inesperado em getAllData()", e);
        if (pageError) {
          pageError.style.display = "block";
          pageError.textContent = "Erro inesperado ao carregar dados (ver console).";
        }
      } finally {
        isFetching = false;
      }
    }

    console.log("[INCHARGE] script.js carregou e iniciou.");

    getAllData();
    setInterval(getAllData, 30000);
    window.addEventListener("resize", fitToViewport);
  });
})();
