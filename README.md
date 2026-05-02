# import java.io.*;
import java.util.*;

public class PlagiarismChecker {

    // Prime number for hashing
    private static final int PRIME = 101;

    // Rabin-Karp string matching
    public static boolean rabinKarp(String text, String pattern) {
        int n = text.length();
        int m = pattern.length();

        if (m > n) return false;

        int patternHash = 0;
        int textHash = 0;
        int h = 1;

        // Calculate h = pow(d, m-1) % PRIME
        for (int i = 0; i < m - 1; i++)
            h = (h * 256) % PRIME;

        // Initial hash values
        for (int i = 0; i < m; i++) {
            patternHash = (256 * patternHash + pattern.charAt(i)) % PRIME;
            textHash = (256 * textHash + text.charAt(i)) % PRIME;
        }

        // Slide pattern over text
        for (int i = 0; i <= n - m; i++) {

            if (patternHash == textHash) {
                // Check characters one by one
                int j;
                for (j = 0; j < m; j++) {
                    if (text.charAt(i + j) != pattern.charAt(j))
                        break;
                }
                if (j == m)
                    return true;
            }

            // Calculate next window hash
            if (i < n - m) {
                textHash = (256 * (textHash - text.charAt(i) * h)
                        + text.charAt(i + m)) % PRIME;

                if (textHash < 0)
                    textHash += PRIME;
            }
        }

        return false;
    }

    // Read file content
    public static String readFile(String path) throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader(path));
        StringBuilder content = new StringBuilder();
        String line;

        while ((line = reader.readLine()) != null) {
            content.append(line).append(" ");
        }
        reader.close();
        return content.toString();
    }

    // Split into sentences
    public static List<String> splitSentences(String text) {
        return Arrays.asList(text.split("[.!?]"));
    }

    public static void main(String[] args) {
        try {
            String file1 = "file1.txt";
            String file2 = "file2.txt";

            String text1 = readFile(file1).toLowerCase();
            String text2 = readFile(file2).toLowerCase();

            List<String> sentences1 = splitSentences(text1);
            List<String> sentences2 = splitSentences(text2);

            System.out.println("Plagiarized Sentences:\n");

            for (String s1 : sentences1) {
                s1 = s1.trim();
                if (s1.length() < 5) continue;

                for (String s2 : sentences2) {
                    s2 = s2.trim();

                    if (rabinKarp(s2, s1)) {
                        System.out.println("- " + s1);
                        break;
                    }
                }
            }

        } catch (IOException e) {
            System.out.println("Error reading files: " + e.getMessage());
        }
    }
}
