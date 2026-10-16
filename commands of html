import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;

public final class Main {
    private static final Path INDEX_FILE = Path.of("index.html").toAbsolutePath().normalize();
    private static final String BOT_INFO = """
            {
              "name": "Discord Moderation Bot",
              "summary": "A lightweight moderator that removes rule-breaking messages, warns the author, and can log incidents to a moderation channel.",
              "checks": [
                {
                  "number": "01",
                  "title": "Blocked language",
                  "description": "Catches configured words, including attempts to disguise them with spaces, numbers, or symbols."
                },
                {
                  "number": "02",
                  "title": "Repeated characters",
                  "description": "Flags a character repeated five or more times in a row."
                },
                {
                  "number": "03",
                  "title": "Long messages",
                  "description": "Removes messages longer than 250 characters."
                },
                {
                  "number": "04",
                  "title": "Links",
                  "description": "Flags messages containing HTTP(S) URLs, Discord invites, or discordapp.com links."
                }
              ],
              "response": [
                "The message is removed.",
                "The author receives a brief warning.",
                "If configured, the incident is sent to the moderation log channel."
              ]
            }
            """;

    private Main() {
    }

    public static void main(String[] args) throws IOException {
        if (args.length > 1) {
            throw new IllegalArgumentException("Usage: java Main [port]");
        }

        int port = args.length == 0 ? 8080 : Integer.parseInt(args[0]);
        if (port < 1 || port > 65535) {
            throw new IllegalArgumentException("Port must be between 1 and 65535.");
        }

        HttpServer server = HttpServer.create(new InetSocketAddress("127.0.0.1", port), 0);
        server.createContext("/", Main::handleRequest);
        server.start();
        System.out.println("Bot website is available at http://127.0.0.1:" + port);
    }

    private static void handleRequest(HttpExchange exchange) throws IOException {
        if (!"GET".equals(exchange.getRequestMethod())) {
            exchange.getResponseHeaders().set("Allow", "GET");
            send(exchange, 405, "text/plain; charset=utf-8", "Method not allowed.");
            return;
        }

        String path = exchange.getRequestURI().getPath();
        if ("/api/bot-info".equals(path)) {
            send(exchange, 200, "application/json; charset=utf-8", BOT_INFO);
            return;
        }

        if (!"/".equals(path) && !"/index.html".equals(path)) {
            send(exchange, 404, "text/plain; charset=utf-8", "Not found.");
            return;
        }

        if (!Files.isRegularFile(INDEX_FILE)) {
            send(exchange, 500, "text/plain; charset=utf-8",
                    "Could not find index.html. Start the server from the bot-website folder.");
            return;
        }

        byte[] html = Files.readAllBytes(INDEX_FILE);
        exchange.getResponseHeaders().set("Content-Type", "text/html; charset=utf-8");
        addSecurityHeaders(exchange);
        exchange.sendResponseHeaders(200, html.length);
        try (var response = exchange.getResponseBody()) {
            response.write(html);
        }
    }

    private static void send(HttpExchange exchange, int status, String contentType, String body)
            throws IOException {
        byte[] bytes = body.getBytes(StandardCharsets.UTF_8);
        exchange.getResponseHeaders().set("Content-Type", contentType);
        addSecurityHeaders(exchange);
        exchange.sendResponseHeaders(status, bytes.length);
        try (var response = exchange.getResponseBody()) {
            response.write(bytes);
        }
    }

    private static void addSecurityHeaders(HttpExchange exchange) {
        exchange.getResponseHeaders().set("X-Content-Type-Options", "nosniff");
        exchange.getResponseHeaders().set("X-Frame-Options", "DENY");
        exchange.getResponseHeaders().set("Referrer-Policy", "no-referrer");
    }
}
