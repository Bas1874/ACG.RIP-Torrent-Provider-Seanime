/// <reference path="./anime-torrent-provider.d.ts" />
/// <reference path="./core.d.ts" />

class Provider {

    private api = "https://acg.rip"

    // Returns the provider settings.
    getSettings(): AnimeProviderSettings {
        return {
            canSmartSearch: false,
            smartSearchFilters: [],
            supportsAdult: true,
            type: "main",
        }
    }

    // Searches for torrents based on the user's query.
    async search(opts: AnimeSearchOptions): Promise<AnimeTorrent[]> {
        const searchUrl = `${this.api}/?term=${encodeURIComponent(opts.query)}`;

        const response = await fetch(searchUrl);
        if (!response.ok) {
            throw new Error(`Failed to fetch search results from acg.rip, status: ${response.status}`);
        }
        const html = await response.text();
        const $ = LoadDoc(html);

        const torrents: AnimeTorrent[] = [];

        $("table.post-index > tbody > tr").each((i, el) => {
            const titleElement = el.find("td.title > span.title > a");
            const dateElement = el.find("td.date time");
            const sizeElement = el.find("td.size");
            const downloadElement = el.find("td.action > a");
            const releaseGroupElement = el.find("td.title > span.label-team > a");

            const name = titleElement.text().trim();
            const link = this.api + titleElement.attr("href");
            const downloadUrl = this.api + downloadElement.attr("href");
            const sizeStr = sizeElement.text().trim();
            const timestamp = dateElement.attr("datetime");
            const releaseGroup = releaseGroupElement.text().trim() || "";

            torrents.push({
                name: name,
                date: this.parseDate(timestamp),
                size: this.parseSize(sizeStr),
                formattedSize: sizeStr,
                seeders: 0, // Not available from the website
                leechers: 0, // Not available from the website
                downloadCount: 0, // Not available from the website
                link: link,
                downloadUrl: downloadUrl,
                releaseGroup: releaseGroup,
                magnetLink: "", 
                infoHash: "",
                resolution: this.parseResolution(name), // <-- ADDED THIS
                isBatch: false,
                episodeNumber: -1,
                isBestRelease: false,
                confirmed: false,
            });
        });

        return torrents;
    }

    // This provider does not support smart search.
    async smartSearch(opts: AnimeSmartSearchOptions): Promise<AnimeTorrent[]> {
        return [];
    }
    
    // This function will get the magnet link from the torrent's detail page.
    async getTorrentMagnetLink(torrent: AnimeTorrent): Promise<string> {
        try {
            const response = await fetch(torrent.link);
            if (!response.ok) { return ""; }
            const html = await response.text();
            const $ = LoadDoc(html);
            const magnetLink = $("a[href^='magnet:']").attr("href");
            return magnetLink || "";
        } catch (error) {
            console.error("Failed to fetch magnet link:", error);
            return "";
        }
    }

    // Get the latest torrents from the homepage.
    async getLatest(): Promise<AnimeTorrent[]> {
        const response = await fetch(this.api);
        if (!response.ok) { return []; }
        const html = await response.text();
        const $ = LoadDoc(html);
        
        const torrents: AnimeTorrent[] = [];
        $("table.post-index > tbody > tr").each((i, el) => {
            const titleElement = el.find("td.title > span.title > a");
            const dateElement = el.find("td.date time");
            const sizeElement = el.find("td.size");
            const downloadElement = el.find("td.action > a");
            const releaseGroupElement = el.find("td.title > span.label-team > a");

            const name = titleElement.text().trim();
            const link = this.api + titleElement.attr("href");
            const downloadUrl = this.api + downloadElement.attr("href");
            const sizeStr = sizeElement.text().trim();
            const timestamp = dateElement.attr("datetime");
            const releaseGroup = releaseGroupElement.text().trim() || "";

            torrents.push({
                name: name,
                date: this.parseDate(timestamp),
                size: this.parseSize(sizeStr),
                formattedSize: sizeStr,
                seeders: 0,
                leechers: 0,
                downloadCount: 0,
                link: link,
                downloadUrl: downloadUrl,
                releaseGroup: releaseGroup,
                magnetLink: "",
                infoHash: "",
                resolution: this.parseResolution(name), // <-- ADDED THIS
                isBatch: false,
                episodeNumber: -1,
                isBestRelease: false,
                confirmed: false,
            });
        });

        return torrents;
    }

    /**
     * Helper function to parse resolution (e.g., "1080p") from a title.
     */
    private parseResolution(title: string): string {
        const match = title.match(/\b(\d{3,4}p)\b/i);
        return match ? match[1] : "";
    }

    /**
     * Helper function to parse a size string (e.g., "89.3 GB") into bytes.
     */
    private parseSize(sizeStr: string): number {
        const sizeMatch = sizeStr.match(/([\d\.]+)\s*(GB|MB|KB)/i);
        if (!sizeMatch) return 0;
        const size = parseFloat(sizeMatch[1]);
        const unit = sizeMatch[2].toUpperCase();
        if (unit === "GB") { return Math.round(size * 1024 * 1024 * 1024); }
        if (unit === "MB") { return Math.round(size * 1024 * 1024); }
        if (unit === "KB") { return Math.round(size * 1024); }
        return 0;
    }

    /**
     * Helper function to convert a Unix timestamp to an ISO 8601 date string.
     */
    private parseDate(timestamp: string): string {
        if (!timestamp) return "";
        const date = new Date(parseInt(timestamp) * 1000);
        return date.toISOString();
    }
}
