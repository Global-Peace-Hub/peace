<script>
    import { onMount } from "svelte";
    import {
        getGeo,
        images,
        sections,
        scrollToSection,
        images_ppl,
    } from "../utils";
    import Map from "../lib/Map.svelte";
    import { navigate } from "svelte5-router";

    let width;
    let all_polygons;
    let showScrollToTop = false; // Visibility of the scroll-to-top button
    let isMenuOpen = false; // Toggle menu visibility
    let isSmallScreen = false; // Check if screen size is small

    // Load GEOJSON
    const json_path = "/data/geojson.json";
    getGeo(json_path).then((geo) => {
        all_polygons = geo;
    });

    // RESEARCH GALLERY
    let imageRowResearch;
    const scrollGalleryResearch = (direction) => {
        if (!imageRowResearch) return;

        const imageWidth = imageRowResearch.scrollWidth / images_ppl.length; // Ensure exact image width scroll
        let targetScroll = imageRowResearch.scrollLeft + direction * imageWidth;

        animateScroll(imageRowResearch, targetScroll, 500); // 500ms smooth scroll
    };

    // PEOPLE GALLERY
    let imageRowPeople;
    function scrollGalleryPeople(direction) {
        if (!imageRowPeople) return;

        const imageWidth = imageRowPeople.scrollWidth / images_ppl.length; // Ensure exact image width scroll
        let targetScroll = imageRowPeople.scrollLeft + direction * imageWidth;

        animateScroll(imageRowPeople, targetScroll, 500); // 500ms smooth scroll
    }

    function animateScroll(element, to, duration) {
        const start = element.scrollLeft;
        const change = to - start;
        const startTime = performance.now();

        function step(currentTime) {
            let progress = (currentTime - startTime) / duration;
            if (progress > 1) progress = 1;

            element.scrollLeft = start + change * easeInOutQuad(progress);

            if (progress < 1) {
                requestAnimationFrame(step);
            }
        }

        requestAnimationFrame(step);
    }

    function easeInOutQuad(t) {
        return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
    }

    // Scroll to top function
    const scrollToTop = () => {
        window.scrollTo({ top: 0, behavior: "smooth" });
    };

    // Check scroll position and update reactivity
    const handleScroll = () => {
        showScrollToTop = window.scrollY > 300; // Trigger button visibility
    };

    function checkScreenSize() {
        isSmallScreen = window.innerWidth < 768; // Adjust breakpoint as needed
    }

    // INIT
    onMount(() => {
        checkScreenSize();
        window.addEventListener("resize", checkScreenSize);

        // Disable automatic scroll restoration
        if ("scrollRestoration" in history) {
            history.scrollRestoration = "manual";
        }

        // Scroll to top on page load
        window.scrollTo({ top: 0, behavior: "auto" });

        // Add event listener for scrolling to top
        window.addEventListener("scroll", handleScroll);
        imageRowResearch = document.querySelector(".image-row-research");
        imageRowPeople = document.querySelector(".image-row-people");

        return () => {
            window.removeEventListener("scroll", handleScroll);
            window.removeEventListener("resize", checkScreenSize);
        };
    });
</script>

<div id="wrapper" bind:clientWidth={width}>
    <div id="home">
        <div class="logos">
            <img alt="University of St Andrews Logo" src="uosa.png" />
            <img alt="University of St Andrews Logo" src="uoe_white.png" />
            <img alt="PeaceRep Logo" src="peacerep_logo.jpg" />
        </div>

        <div id="navigation">
            <!-- Hamburger Icon (Mobile) -->
            <i
                style="padding: 5px;"
                class="fa fa-bars menu-icon"
                on:click={() => (isMenuOpen = !isMenuOpen)}
            ></i>

            <!-- Individual Buttons (Desktop) -->
            {#if !isSmallScreen}
                {#each sections as section}
                    <button
                        class="menu-button"
                        on:click={() => scrollToSection(section.id)}
                    >
                        {section.name}
                    </button>
                {/each}
            {/if}

            <!-- Dropdown Menu (Mobile) -->
            {#if isSmallScreen && isMenuOpen}
                <ul class="dropdown">
                    {#each sections as section}
                        <li on:click={() => scrollToSection(section.id)}>
                            {section.name}
                        </li>
                    {/each}
                </ul>
            {/if}
        </div>

        <!-- <h1>MEND: Mediation Events and Negotiators Database</h1> -->
        <h1>Global PeaceHub</h1>
        <h3>
            Understanding shifts in the geopolitical context of peace and
            transition processes
        </h3>
    </div>
    <main
        id="map"
        bind:clientWidth={width}
        style="height: calc(var(--vh, 1vh) * 100);"
    >
        <h3
            style="
      z-index: 400; 
      position: absolute; 
      width: 100px;
      text-align: left; 
      font-size: 20px;
      border-radius: 2px"
        >
            Explore
        </h3>
        <Map
            {all_polygons}
            inflate={(data) => {
                navigate("/" + data);
                // window.open("/glopad/" + data, "_blank");
            }}
        />
    </main>

    <div id="people">
        <h3
            style="width: 100px;text-align: left;border-radius: 2px; font-size:20px;"
        >
            Team
        </h3>
        <div class="image-grid">
            {#each images_ppl as { src, name, position } (src)}
                <div class="image-container-people">
                    <img loading="lazy" {src} alt={name} />
                    <p>{name}</p>
                    <p>{position}</p>
                </div>
            {/each}
        </div>
    </div>

    <!-- <div id="people">
        <h3
            style="width: 100px;text-align: left;border-radius: 2px; font-size:20px;"
        >
            Team
        </h3>
        <div class="gallery-container-people">
            <button
                class="arrow left"
                style="border: 2px solid white;"
                aria-label="Scroll left"
                on:click={() => scrollGalleryPeople(-1)}
            >
                <i
                    class="fa fa-arrow-left"
                    style="color: white;"
                    aria-hidden="true"
                ></i>
            </button>
            <div class="image-row-people">
                {#each images_ppl as { src, name, position } (src)}
                    <div class="image-container">
                        <img loading="lazy" {src} alt={name} />
                        <p>{name}</p>
                        <p>{position}</p>
                    </div>
                {/each}
            </div>
            <button
                class="arrow right"
                style="border: 2px solid white;"
                aria-label="Scroll right"
                on:click={() => scrollGalleryPeople(1)}
            >
                <i
                    class="fa fa-arrow-right"
                    style="color: white;"
                    aria-hidden="true"
                ></i>
            </button>
        </div>
    </div> -->

    <!-- research section -->
    <div id="research">
        <h3
            style="width: 100px;text-align: left;border-radius: 2px; font-size:20px;"
        >
            Research
        </h3>
        <div class="gallery-container-research">
            <button
                class="arrow left"
                style="border: 2px solid white;"
                aria-label="Scroll left"
                on:click={() => scrollGalleryResearch(-1)}
            >
                <i
                    class="fa fa-arrow-left"
                    style="color: white;"
                    aria-hidden="true"
                ></i>
            </button>
            <div class="image-row-research">
                {#each images as { src, name } (src)}
                    <div class="image-container">
                        <img loading="lazy" {src} alt={name} />
                        <p>{name}</p>
                    </div>
                {/each}
            </div>
            <button
                class="arrow right"
                aria-label="Scroll right"
                style="border: 2px solid white;"
                on:click={() => scrollGalleryResearch(1)}
            >
                <i
                    class="fa fa-arrow-right"
                    style="color: white;"
                    aria-hidden="true"
                ></i>
            </button>
        </div>

        <div id="publications">
            <h3>2025</h3>
            <p>
                Elisa D’Amico (2025) Conflict in a Warming World: How Climate
                Shocks Impact Rebel Demands and Peace Agreement Outcomes.
                [Climate & Natural Resources Series]. PeaceRep: The Peace and
                Conflict Resolution Evidence Platform, University of Edinburgh.
            </p>
            <h3>2024</h3>
            <p>
                Mateja Peter (2024) ‘Global fragmentation and collective
                security instruments: Weakening the liberal international order
                from within.’ Politics and Governance, 12 (7357), 1-15.
            </p>
            <p>
                Sanja Badanjak and Mateja Peter (2024) ‘Diversification and
                Congestion in Peacemaking: What the Data Says’. in Theresa
                Whitfield (ed.), Still time to talk: Adaptation and innovation
                in peace mediation, Accord 30. London: Conciliation Resources.
            </p>
            <p>
                Mateja Peter and Kasia Houghton (2024) Russia and China in
                Liberal Peacebuilding: Perceptions and Engagement with the
                Dayton Peace Process in Bosnia and Herzegovina. [Global
                Transitions Series]. PeaceRep: The Peace and Conflict Resolution
                Evidence Platform, University of Edinburgh.
            </p>
            <p>
                Niamh Henry (2024) Extracting Named Actors from Text: Using
                Named Entity Recognition (NER) in Peace and Conflict Studies
                [Peace Analytics Series]. PeaceRep: The Peace and Conflict
                Resolution Evidence Platform, University of Edinburgh.
            </p>
            <p>
                Elisa D’Amico (2024) Semi-Automated Coding for Conflict
                Mediation Research: Database Development. Peace Analytics
                Series, University of Edinburgh: Peace and Conflict Resolution
                Evidence Platform: November 2024.
            </p>
            <p>
                Kasia Houghton (2024). ‘The Competition over Norms: The Case of
                the Syrian Conflict.’ In Benjamin Houghton and Kasia Houghton
                (eds.) China, Russia and the USA in the Middle East: The Contest
                for Supremacy. London: Routledge, 46-
            </p>
            <p>
                Mateja Peter and Ruoxi Wang (2024) ‘China’s Approach to Human
                Security within and outside UN Peacekeeping: Drawing Lessons
                from South Sudan.’ Journal of International Peacekeeping, 27(1),
                85-114.
            </p>
            <h3>2023</h3>
            <p>
                Mateja Peter and Kasia Houghton (2023) Congestion and
                Diversification of Third-Party Mediation in Sudan and South
                Sudan: First Look at some Longer-Term Trends. [Global
                Transitions Series]. PeaceRep: The Peace and Conflict Resolution
                Evidence Platform, University of Edinburgh.
            </p>
            <p>
                Mateja Peter and Marcel Plichta (2023) China and Russia in
                Sudan: Surveying data on economic and military engagement.
                [Global Transitions Series]. PeaceRep: The Peace and Conflict
                Resolution Evidence Platform, University of Edinburgh.th Sudan.’
                Journal of International Peacekeeping, 27(1), 85-114.
            </p>
            <p>
                Sanja Badanjak (2023) Third Parties in Peace Agreements: First
                Look at New Data and Key Trends. [Global Transitions Series].
                PeaceRep: The Peace and Conflict Resolution Evidence Platform,
                University of Edinburgh.
            </p>
            <p>
                Sanja Badanjak (2023) EU External Action and Development
                Spending in a Time of Covid-19. [Covid-19 Series]. PeaceRep: The
                Peace and Conflict Resolution Evidence Platform, University of
                Edinburgh.
            </p>
            <h3>2022</h3>
            <p>
                Mateja Peter and Haley Rice (2022) Non-Western approaches to
                peacemaking and peacebuilding: State-of-the-art and an agenda
                for research. [Global Transitions Series]. PeaceRep: The Peace
                and Conflict Resolution Evidence Platform, University of
                Edinburgh.
            </p>
            <h3>Global Transitions Series</h3>
            <p>
                As part of the PeaceRep consortium, the project also produces
                the <a
                    target="_blank"
                    href="https://peacerep.org/research/geopolitical-transitions/"
                    >Global Transitions</a
                > series, edited by Mateja Peter. This includes publications by the
                core team and further studies produced in the wider consortium.
            </p>
        </div>
    </div>

    <div id="about">
        <h3
            style="width: 100px;text-align: left;border-radius: 2px; font-size:20px;"
        >
            About
        </h3>
        <div id="about_content">
            <p>
                Our research explores fragmentations in the global order and how
                these impact peace and transition processes. We seek to better
                understand why and how different third-party actors – state,
                intergovernmental, and non-governmental actors – intervene in
                attempts to broker peace, and how they see themselves
                contributing to reduction of conflict and risks of conflict
                relapse. We also study how local actors are navigating this
                multiplicity of mediators and peacebuilders and how this is
                shaping conflict outcomes and post-conflict governance.
            </p>
            <p>
                The project produces dedicated case and trends studies as part
                of the PeaceRep Global Transitions series and is developing two
                complementary datasets.
            </p>
            <p style="margin-left: 30px; margin-right: 30px;">
                • Third Parties in Peace Agreements dataset. Most formal
                agreements in peace processes are supported by parties external
                to the armed conflict, in the form of third-party signature to
                the agreement. In the PA-X collection of peace agreements that
                address inter- and intra- state, or mixed conflicts, around
                sixty percent of agreements feature a signatory which is not one
                of the main conflict parties, and around one-third of all
                agreements feature an international third-party signatory. The
                Third Parties in Peace Agreements dataset draws on the Peace
                Agreement Actors Dataset (PAA-X) and is the only existing data
                resource on third-party signatories in peace agreements. It
                allows for an in-depth examination of the patterns of engagement
                in peace agreements and provides valuable insights into what
                actors are necessary for conflict parties to commit to a formal,
                written agreement. This data is instrumental in discerning the
                regional and topical priorities of individual interveners since
                1990, giving us insight into the types of agreements supported
                by individual third parties.
            </p>
            <p style="margin-left: 30px; margin-right: 30px;">
                • Mediation Event and Negotiators Database (MEND) is a new and
                growing resource for researchers and practitioners with a focus
                on peace and conflict resolution. The dataset comprehensively
                covers broader peacemaking efforts within major armed conflicts,
                tracking all mediation and mediation-related events involving
                external third-party actors, regardless of whether these events
                result in a formal peace agreement. It captures mediation as
                part of formal peace initiatives and mediation efforts running
                in support or in parallel to these. Each record corresponds to a
                unique event, providing detailed metadata about location, third
                parties, local actors, and individuals involved. The MEND
                dataset facilitates a nuanced understanding of the roles various
                actors play in contributing to the peace processes and in
                brokering agreements. It also enables the identification of
                unsuccessful or spoiler mediation efforts that may still have
                influenced the ongoing conflict management. In an era marked by
                global fragmentation, this data plays a crucial role in
                providing insights into where, when, how, and why actors engage
                in mediation activities, allowing for the mapping of network
                dynamics between international and local actors, and
                identification of the mediation conditions conducive to the
                eventual signing of agreements.
            </p>
            <h3>Funding</h3>
            <p>
                Our research is supported by the Peace and Conflict Resolution
                Evidence Platform (PeaceRep), funded by UK International
                Development from the UK government. However, the views expressed
                are those of the authors and do not necessarily reflect the UK
                government’s official policies. We receive additional support
                from the Universities of St Andrews and Edinburgh.
            </p>
        </div>
    </div>

    <!-- Scroll to Top Button -->
    <!-- {#if showScrollToTop}
        <button
            id="scrollToTop"
            on:click={scrollToTop}
            aria-label="Scroll to top"
        >
            <i class="fa fa-arrow-up" style="color: yellow;" aria-hidden="true"
            ></i>
        </button>
    {/if} -->
</div>

<style>
    /* General styling */
    #wrapper {
        width: 100%;
    }

    .logos {
        display: flex;
        gap: 5px;
        position: absolute;
        top: 5px;
        right: 5px;
    }

    img {
        height: 30px;
        margin-left: 5px;
    }

    h1 {
        color: white;
        font-weight: 800;
        margin-bottom: 5px;
    }

    h3 {
        color: white;
        font-weight: 500;
        padding: 10px;
        margin: 0px;
    }

    #home {
        position: relative;
        width: 100%;
        height: 300px;
        align-content: center;
        text-align: center;

        background-color: #001c23;
    }

    main {
        width: 100%;
        display: flex;
        overflow: hidden;
        text-align: center;
    }

    #navigation {
        position: absolute;
        top: 0px;
        display: flex;
        align-items: center;
        gap: 2px;
    }

    .menu-button {
        font-family: "Montserrat", sans-serif;
        background: #252525;
        border-radius: 2px;
        border: none;
        padding: 5px;
        width: 110px;
        font-size: 16px;
        cursor: pointer;
        color: white;
    }

    .menu-button:hover {
        background: rgb(78, 78, 78);
        color: white;
    }

    .dropdown {
        list-style: none;
        position: absolute;
        background: black;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        padding: 10px;
        top: 30px;
        left: 5px;
        display: flex;
        flex-direction: column;
        gap: 10px;
    }

    .menu-icon {
        display: none;
        cursor: pointer;
        font-size: 24px;
    }

    @media (max-width: 767px) {
        .menu-button {
            display: none;
        }

        .menu-icon {
            display: block;
        }
    }

    #scrollToTop {
        position: fixed;
        bottom: 20px;
        left: 20px;
        background-color: #000000;
        color: white;
        border: none;
        border-radius: 50%;
        width: 50px;
        height: 50px;
        display: flex;
        justify-content: center;
        align-items: center;
        cursor: pointer;
        box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.3);
        z-index: 1000;
    }

    #scrollToTop:hover {
        background-color: #042645;
    }

    #scrollToTop i {
        font-size: 20px;
    }

    #research,
    #people,
    #about {
        position: relative;
    }

    #about_content {
        position: relative;
        margin: auto;
        width: 80%;
    }

    #research {
        background-color: #003645;
    }
    #people {
        background-color: #001c23;
    }
    #about {
        position: relative;
        background-color: #001c23;
        padding-bottom: 20px;
        font-weight: 200;
    }
    #publications {
        position: relative;
        margin: auto;
        width: 80%;
        padding: 20px;
        font-weight: 200;
    }

    .gallery-container-research,
    .gallery-container-people {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: space-between;
        overflow: hidden;
        width: 100%;
        height: 70vh;
        padding-top: 50px;
        padding-bottom: 50px;
    }

    .gallery-container-research {
        background-color: #003645;
    }
    .gallery-container-people {
        background-color: #001c23;
    }

    .image-row-research,
    .image-row-people {
        display: flex;
        overflow-x: auto;
        scroll-behavior: smooth;
        scrollbar-width: none; /* Hide scrollbar for Firefox */
        -ms-overflow-style: none; /* Hide scrollbar for IE/Edge */
        width: 100%;
        height: 100%;
        align-items: center; /* Center items vertically */
    }

    .image-row-research::-webkit-scrollbar {
        display: none; /* Hide scrollbar for Chrome, Safari, and Opera */
    }

    .image-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center; /* Center image and text vertically within the container */
        margin: 0 8px;
        height: 100%; /* Ensures it takes up the full height of the row */
    }

    .image-container img {
        height: 100%;
        object-fit: cover;
        border-radius: 3px;
    }

    .image-container p {
        margin: 10px 0 0;
        font-size: 1rem;
        color: white;
        text-align: center;
        line-height: 1.2;
    }

    #people {
        text-align: center;
    }

    .image-grid {
        display: grid;
        gap: 20px;
        width: 90%;
        max-width: 1200px;
        margin: auto;

        /* Default: 1 column (mobile first) */
        grid-template-columns: 1fr;
    }

    /* Medium screens: 2 rows of 3 */
    @media (min-width: 600px) {
        .image-grid {
            grid-template-columns: repeat(3, 1fr);
        }
    }

    /* Large screens: 1 row of 6 */
    @media (min-width: 1024px) {
        .image-grid {
            grid-template-columns: repeat(6, 1fr);
        }
    }

    .image-container-people {
        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
        padding-top: 80px;
        padding-bottom: 80px;
    }

    .image-container-people img {
        width: 100%;
        height: auto;
        border-radius: 3px;
    }

    .image-container-people p {
        margin: 10px 0 0;
        font-size: 13px;
        color: white;
        line-height: 1;
    }

    .arrow {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        background: rgb(0, 0, 0);
        color: white;
        border: none;
        border-radius: 50%;
        width: 50px;
        height: 50px;
        font-size: 18px;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        z-index: 10;
    }

    .arrow.left {
        left: 10px;
    }

    .arrow.right {
        right: 10px;
    }

    .arrow:hover {
        background: rgba(0, 0, 0, 0.8);
    }
</style>
