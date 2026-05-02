
import java.io.*;
import java.util.*;

public class PlagiarismChecker {

    // Large prime for Rabin-Karp
    private static final int PRIME = 1000000007;

    // Rabin-Karp algorithm (kept for assignment requirement)
    public static boolean rabinKarp(String text, String pattern) {
        int n = text.length();
        int m = pattern.length();

        if (m > n) {
            return false;
        }

        long patternHash = 0, textHash = 0, h = 1;

        for (int i = 0; i < m - 1; i++) {
            h = (h * 256) % PRIME;
        }

        for (int i = 0; i < m; i++) {
            patternHash = (256 * patternHash + pattern.charAt(i)) % PRIME;
            textHash = (256 * textHash + text.charAt(i)) % PRIME;
        }

        for (int i = 0; i <= n - m; i++) {

            if (patternHash == textHash) {
                int j;
                for (j = 0; j < m; j++) {
                    if (text.charAt(i + j) != pattern.charAt(j)) {
                        break;
                    }
                }
                if (j == m) {
                    return true;
                }
            }

            if (i < n - m) {
                textHash = (256 * (textHash - text.charAt(i) * h)
                        + text.charAt(i + m)) % PRIME;

                if (textHash < 0) {
                    textHash += PRIME;
                }
            }
        }
        return false;
    }

    // Read file
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
        return Arrays.asList(text.split("[.!?]+\\s*"));
    }

    // Clean text (important)
    public static String clean(String s) {
        return s.toLowerCase()
                .replaceAll("[^a-z0-9 ]", "")
                .trim()
                .replaceAll("\\s+", " ");
    }

    public static void main(String[] args) {
        System.out.println(new File("file1.txt").getAbsolutePath());
        try {
            String file1 = "file1.txt";
           String file2 = "file2.txt";
            

            String text1 = readFile(file1);
            String text2 = readFile(file2);

            System.out.println("DEBUG: Content of File 1: [" + text1 + "]");
            System.out.println("DEBUG: Content of File 2: [" + text2 + "]");

            List<String> sentences1 = splitSentences(text1);
            List<String> sentences2 = splitSentences(text2);

            System.out.println("Plagiarized Sentences:\n");

            boolean found = false;

            for (String s1 : sentences1) {

                //s1 = clean(s1);
                if (s1.length() < 10) {
                    continue;
                }

                for (String s2 : sentences2) {

                    //s2 = clean(s2);
                    // Reliable detection
                    if (s2.contains(s1) || s1.contains(s2) || rabinKarp(s2, s1)) {
                        System.out.println("- " + s1);
                        found = true;
                        break;
                    }
                }
            }

            if (!found) {
                System.out.println("No plagiarism detected.");
            }

        } catch (IOException e) {
            System.out.println("Error reading files: " + e.getMessage());
        }
    }
}
