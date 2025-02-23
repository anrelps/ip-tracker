<script lang="ts">
    import '@picocss/pico';
    import { onMount } from 'svelte';

    let map: any;
    let L: any;
    let ip: string;
    let isp: string;
    let location: string;
    let timezone: string;
    let latitude: number;
    let longitude: number;
    const apiKey = import.meta.env.VITE_IPIFY_API_KEY;
    if (!apiKey) {
        console.error('API key not found! Please check your .env file');
    }
    let searchInput: string = '';
    let marker: any;
    let error: string = '';
    let isLoading: boolean = false;

    async function searchLocation(ipAddress: string = '') {
        error = '';
        isLoading = true;

        try {
            const url = ipAddress
                ? `https://geo.ipify.org/api/v2/country,city?apiKey=${apiKey}&ipAddress=${ipAddress}`
                : `https://geo.ipify.org/api/v2/country,city?apiKey=${apiKey}`;

            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            const data = await response.json();

            if (data.code) {
                throw new Error(
                    data.messages || 'Invalid IP address or domain'
                );
            }

            ip = data.ip;
            location = data.location.city;
            timezone = data.location.timezone;
            latitude = data.location.lat;
            longitude = data.location.lng;
            isp = data.isp;

            if (map) {
                map.setView([latitude, longitude], 13);
                if (marker) {
                    marker.remove();
                }
                marker = L.marker([latitude, longitude]).addTo(map);
            }
        } catch (err) {
            error = err.message || 'An error occurred while fetching the data';
            console.error('Error:', err);
        } finally {
            isLoading = false;
        }
    }

    function handleSubmit(event: Event) {
        event.preventDefault();
        if (!searchInput.trim()) {
            error = 'Please enter an IP address or domain';
            return;
        }
        searchLocation(searchInput.trim());
    }

    onMount(async () => {
        await new Promise((resolve) => {
            if (window.L) {
                resolve(true);
            } else {
                window.addEventListener('load', () => resolve(true), {
                    once: true,
                });
            }
        });

        L = window.L;
        map = L.map('map');
        L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution:
                '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
        }).addTo(map);

        await searchLocation();
    });
</script>

<svelte:head>
    <link
        rel="stylesheet"
        href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
        integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
        crossorigin=""
    />
    <script
        src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
        integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
        crossorigin=""
    ></script>
</svelte:head>

<main>
    <div class="container">
        <h1>IP Address Tracker</h1>
        <form class="border" on:submit={handleSubmit}>
            <input
                type="text"
                bind:value={searchInput}
                placeholder="Search for any IP address or domain"
                aria-label="IP address or domain"
                disabled={isLoading}
            />
            <button type="submit" aria-label="Search" disabled={isLoading}>
                <svg
                    fill="white"
                    height="24px"
                    width="24px"
                    version="1.1"
                    id="Layer_1"
                    xmlns="http://www.w3.org/2000/svg"
                    xmlns:xlink="http://www.w3.org/1999/xlink"
                    viewBox="0 0 330 330"
                    xml:space="preserve"
                >
                    <path
                        id="XMLID_222_"
                        d="M250.606,154.389l-150-149.996c-5.857-5.858-15.355-5.858-21.213,0.001
             c-5.857,5.858-5.857,15.355,0.001,21.213l139.393,139.39L79.393,304.394c-5.857,5.858-5.857,15.355,0.001,21.213
             C82.322,328.536,86.161,330,90,330s7.678-1.464,10.607-4.394l149.999-150.004c2.814-2.813,4.394-6.628,4.394-10.606
             C255,161.018,253.42,157.202,250.606,154.389z"
                    />
                </svg>
            </button>
        </form>
        {#if error}
            <p class="error">{error}</p>
        {/if}
        <div class="grid">
            <div class="separator">
                <h2>IP Address</h2>
                <p class="ip">{ip}</p>
            </div>
            <div class="separator">
                <h2>Location</h2>
                <p>{location}</p>
            </div>
            <div class="separator">
                <h2>Timezone</h2>
                <p>{timezone}</p>
            </div>
            <div>
                <h2>ISP</h2>
                <p>{isp}</p>
            </div>
        </div>
    </div>
    <div id="map"></div>
    <div class="github">
        <a target="_blank" href="https://github.com/anrelps"
            >anrelps - github<span
                ><svg
                    id="Capa_1"
                    enable-background="new 0 0 512 512"
                    viewBox="0 0 512 512"
                    xmlns="http://www.w3.org/2000/svg"
                    ><g
                        ><circle cx="256" cy="256" r="256" /><g fill="#fff"
                            ><path
                                clip-rule="evenodd"
                                d="m256 64.5c-108.425 0-196.348 87.904-196.348 196.348 0 86.751 56.259 160.35 134.275 186.314 9.812 1.816 13.415-4.261 13.415-9.447 0-4.68-.181-20.15-.267-36.556-54.624 11.879-66.15-23.167-66.15-23.167-8.932-22.694-21.801-28.73-21.801-28.73-17.816-12.186 1.343-11.936 1.343-11.936 19.715 1.384 30.098 20.236 30.098 20.236 17.511 30.015 45.933 21.338 57.139 16.321 1.762-12.688 6.849-21.354 12.466-26.256-43.612-4.963-89.46-21.801-89.46-97.036 0-21.436 7.672-38.952 20.233-52.702-2.039-4.947-8.76-24.916 1.902-51.963 0 0 16.489-5.274 54.008 20.129 15.664-4.351 32.461-6.533 49.146-6.609 16.686.076 33.496 2.258 49.188 6.609 37.477-25.404 53.941-20.129 53.941-20.129 10.688 27.047 3.966 47.016 1.927 51.963 12.59 13.75 20.207 31.267 20.207 52.702 0 75.416-45.933 92.019-89.656 96.88 7.043 6.094 13.317 18.042 13.317 36.36 0 26.272-.225 47.416-.225 53.884 0 5.226 3.534 11.349 13.485 9.421 77.975-25.993 134.164-99.566 134.164-186.289.001-108.443-87.909-196.347-196.347-196.347z"
                                fill-rule="evenodd"
                            /><path
                                d="m134.02 346.411c-.432.978-1.969 1.27-3.366.6-1.426-.641-2.223-1.972-1.762-2.95.422-1.003 1.959-1.283 3.382-.613 1.425.642 2.238 1.986 1.746 2.963z"
                            /><path
                                d="m141.974 355.283c-.937.867-2.769.464-4.01-.908-1.286-1.368-1.524-3.201-.575-4.08.965-.867 2.74-.46 4.026.908 1.286 1.384 1.537 3.201.559 4.08z"
                            /><path
                                d="m149.715 366.59c-1.204.838-3.172.054-4.388-1.692-1.204-1.746-1.204-3.842.028-4.68 1.216-.838 3.156-.082 4.389 1.648 1.2 1.774 1.2 3.87-.029 4.724z"
                            /><path
                                d="m160.32 377.516c-1.077 1.187-3.369.867-5.046-.753-1.718-1.581-2.194-3.829-1.117-5.017 1.092-1.188 3.397-.854 5.087.752 1.705 1.582 2.223 3.843 1.076 5.018z"
                            /><path
                                d="m174.952 383.86c-.476 1.537-2.683 2.235-4.906 1.582-2.223-.673-3.674-2.474-3.226-4.026.46-1.55 2.68-2.277 4.918-1.578 2.22.669 3.674 2.457 3.214 4.022z"
                            /><path
                                d="m191.022 385.035c.054 1.619-1.832 2.962-4.166 2.991-2.347.054-4.248-1.257-4.274-2.852 0-1.635 1.845-2.962 4.191-3.004 2.334-.044 4.249 1.258 4.249 2.865z"
                            /><path
                                d="m205.974 382.492c.279 1.578-1.343 3.201-3.661 3.633-2.28.419-4.388-.559-4.68-2.124-.283-1.619 1.368-3.242 3.645-3.661 2.321-.404 4.401.545 4.696 2.152z"
                            /></g
                        ></g
                    ><g /><g /><g /><g /><g /><g /><g /><g /><g /><g /><g /><g
                    /><g /><g /><g /></svg
                ></span
            ></a
        >
    </div>
</main>

<style is:global>
    main {
        background-image: radial-gradient(
            circle farthest-corner at 10% 20%,
            rgb(7, 62, 163) 16.3%,
            rgb(1, 26, 73) 100.2%
        );
    }

    h1 {
        color: var(--pico-primary-inverse);
        padding-top: 10rem;
        text-align: center;
        margin-bottom: 0.25rem;
    }

    h2 {
        text-transform: uppercase;
        font-size: calc(var(--pico-font-size) - 1.1rem);
        color: var(--pico-muted-color);
    }

    form {
        display: flex;
        justify-content: center;
        align-items: center;
        max-width: 750px;
        margin: 0 auto;
        padding-inline: 2rem;
    }

    form button {
        margin-left: 1rem;
        flex-basis: 24px;
        flex-shrink: 0;
        background-color: #367bfc;
        border: 1px solid white;
    }

    .grid {
        position: relative;
        padding-block: 2rem;
        padding-inline: 3rem;
        background-color: var(--pico-background-color);
        border-radius: 1rem;
        box-shadow: var(--pico-box-shadow);
        z-index: 999;
        margin-bottom: -15vh;
        border: 1px solid rgba(0, 0, 0, 0.3);
    }

    .grid p {
        font-size: calc(var(--pico-font-size) + 0.1rem);
        font-weight: 700;
        color: var(--pico-h1-color);
    }

    .separator {
        border-right: 0;
        padding-right: 1rem;
    }

    .github {
        text-align: center;
        background-color: var(--pico-background-color);
        padding-block: 2rem;
    }

    .github svg {
        margin-left: 0.5rem;
        margin-bottom: 0.25rem;
        width: 24px;
        height: 24px;
    }

    #map {
        width: 100%;
        height: 60vh;
        z-index: 1;
        box-shadow: var(--pico-box-shadow);
        border: 1px solid rgba(0, 0, 0, 0.3);
    }

    @media (min-width: 768px) {
        .grid {
            margin-bottom: -7vh;
        }
        .separator {
            border-right: 1px solid var(--pico-muted-color);
            padding-right: 1rem;
        }
    }

    :global(.leaflet-marker-icon) {
        background: none;
        border: none;
        outline: none;
        box-shadow: none;
        padding: 0;
        margin: 0;
    }

    :global(.leaflet-control-container *) {
        line-height: 0;
        padding: 0;
        margin: 0;
        border: none;
    }

    .error {
        color: red;
        text-align: center;
        margin-top: 1rem;
        font-weight: bold;
        background-color: var(--pico-background-color);
        padding: 0.5rem;
        border-radius: 0.3rem;
        max-width: 750px;
        margin-inline: auto;
    }

    input:disabled,
    button:disabled {
        cursor: not-allowed;
        opacity: 0.7;
    }
</style>
