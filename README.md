# CSVProcessor
# Project Name:
CSVProcessor - Multi-Threaded CSV File Processor

Project Description:
CSVProcessor is a multi-threaded Java application designed to efficiently read and process large CSV files using ExecutorService. The project enhances performance by executing parallel data processing, making it suitable for handling large datasets.

🚀 Features:

Parallel processing of CSV lines for improved speed.
Utilizes Java Multi-Threading for optimized performance.
Supports Batch Processing for efficient data handling.
Can be extended to use Spring Batch for large-scale ETL processing.



import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class CSVProcessor {

    private static final int THREAD_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int BATCH_SIZE = 100; // عدد السطور في كل batch

    public static void main(String[] args) {
        String filePath = "data.csv";
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        CountDownLatch latch = null;

        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            String line;
            List<String> batch = new ArrayList<>();
            List<Future<?>> futures = new ArrayList<>();

            // تخطي الـ headers
            br.readLine();

            while ((line = br.readLine()) != null) {
                batch.add(line);

                // لما نوصل للـ batch size، نعالج الدفعة
                if (batch.size() == BATCH_SIZE) {
                    List<String> batchCopy = new ArrayList<>(batch);
                    Future<?> future = executor.submit(() -> processBatch(batchCopy));
                    futures.add(future);
                    batch.clear();
                }
            }

            // معالجة أي بيانات متبقية
            if (!batch.isEmpty()) {
                List<String> batchCopy = new ArrayList<>(batch);
                futures.add(executor.submit(() -> processBatch(batchCopy)));
            }

            // انتظار إنهاء كل الـ futures
            for (Future<?> f : futures) f.get();

            System.out.println("✅ Done processing CSV in parallel batches.");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executor.shutdown();
        }
    }

    // معالجة دفعة كاملة من السطور
    private static void processBatch(List<String> batch) {
        for (String line : batch) {
            processLine(line);
        }
    }

    // معالجة سطر واحد
    private static void processLine(String line) {
        String[] parts = line.split(",");
        // منطق المعالجة – مثال:
        System.out.println(Thread.currentThread().getName() + " -> " + Arrays.toString(parts));

        // ممكن تضيف هنا منطق حفظ في قاعدة بيانات، فلترة، تحويل بيانات، إلخ.
    }
}

