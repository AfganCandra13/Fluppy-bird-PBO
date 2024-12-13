import javafx.animation.KeyFrame;
import javafx.animation.Timeline;
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.input.KeyCode;
import javafx.scene.layout.Pane;
import javafx.scene.paint.Color;
import javafx.scene.shape.Rectangle;
import javafx.scene.text.Text;
import javafx.stage.Stage;
import javafx.util.Duration;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class FlappyBirdGame extends Application {
    private static final double WIDTH = 600;
    private static final double HEIGHT = 400;
    private static final double BIRD_SIZE = 20;
    private static final double PIPE_GAP = 150;
    private static final double PIPE_WIDTH = 50;
    private static final double GRAVITY = 0.5;
    private static final double FLAP_STRENGTH = -10;

    private double birdY = HEIGHT / 2;
    private double birdVelocity = 0;
    private Rectangle bird;
    private List<Rectangle> pipes;
    private Pane pane;
    private Text scoreText, gameOverText, restartText, startText;
    private Timeline gameLoop, pipeSpawner;
    private boolean gameStarted = false;
    private int score = 0;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        pane = new Pane();
        Scene scene = new Scene(pane, WIDTH, HEIGHT);
        scene.setFill(Color.CYAN);

        // Inisialisasi elemen permainan
        initializeGameElements();

        // Event handler untuk tombol
        scene.setOnKeyPressed(event -> {
            if (!gameStarted && event.getCode() == KeyCode.SPACE) {
                startGame();
            } else if (gameStarted && event.getCode() == KeyCode.SPACE) {
                birdVelocity = FLAP_STRENGTH; // Burung terbang ke atas
            } else if (!gameStarted && event.getCode() == KeyCode.TAB) {
                restartGame();
            }
        });

        primaryStage.setTitle("Flappy Bird Game");
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private void initializeGameElements() {
        // Reset variabel permainan
        birdY = HEIGHT / 2;
        birdVelocity = 0;
        pipes = new ArrayList<>();
        score = 0;
        gameStarted = false;

        // Reset tampilan
        pane.getChildren().clear();

        // Burung
        bird = new Rectangle(BIRD_SIZE, BIRD_SIZE, Color.YELLOW);
        bird.setX(100);
        bird.setY(birdY);

        // Teks skor
        scoreText = new Text(10, 20, "Score: 0");
        scoreText.setFill(Color.BLACK);

        // Teks "Game Over"
        gameOverText = new Text(WIDTH / 2 - 100, HEIGHT / 2, "");
        gameOverText.setFill(Color.BLACK);

        // Teks "Press TAB to Restart"
        restartText = new Text(WIDTH / 2 - 100, HEIGHT / 2 + 30, "");
        restartText.setFill(Color.BLACK);

        
        startText = new Text(WIDTH / 2 - 150, HEIGHT / 2 - 30, "Tekan Spasi untuk Memulai dan terbang");
        startText.setFill(Color.BLACK);

        // Tambahkan elemen ke pane
        pane.getChildren().addAll(bird, scoreText, gameOverText, restartText, startText);
    }

    private void startGame() {
        gameStarted = true;

        // Bersihkan teks game over, restart, dan start
        gameOverText.setText("");
        restartText.setText("");
        startText.setText("");

        // Loop utama permainan
        gameLoop = new Timeline(new KeyFrame(Duration.millis(20), e -> gameUpdate()));
        gameLoop.setCycleCount(Timeline.INDEFINITE);
        gameLoop.play();

        // Loopbuat pipa
        pipeSpawner = new Timeline(new KeyFrame(Duration.seconds(2), e -> spawnPipes()));
        pipeSpawner.setCycleCount(Timeline.INDEFINITE);
        pipeSpawner.play();
    }

    private void gameUpdate() {
    // Update posisi burung
    birdVelocity += GRAVITY;
    birdY += birdVelocity;
    bird.setY(birdY);

    // Cek apakah burung 
    if (birdY > HEIGHT - BIRD_SIZE || birdY < 0) {
        endGame();
        return;
    }

    // Update posisi pipa
    List<Rectangle> pipesToRemove = new ArrayList<>();
    for (int i = 0; i < pipes.size(); i += 2) {
        Rectangle topPipe = pipes.get(i);
        Rectangle bottomPipe = pipes.get(i + 1);

        topPipe.setX(topPipe.getX() - 3);
        bottomPipe.setX(bottomPipe.getX() - 3);

        // Tambah skor pas burung melewati pasangan pipa
        if (topPipe.getX() + PIPE_WIDTH < bird.getX() && topPipe.getUserData() != null && topPipe.getUserData().equals("not_scored")) {
            score++;
            topPipe.setUserData("scored"); // Tandai bahwa pipa telah dilewati
            bottomPipe.setUserData("scored");
            scoreText.setText("Score: " + score);
        }

        // Hapus pipa jika keluar layar
        if (topPipe.getX() + PIPE_WIDTH < 0) {
            pipesToRemove.add(topPipe);
            pipesToRemove.add(bottomPipe);
        }
    }

    pipes.removeAll(pipesToRemove);
    pane.getChildren().removeAll(pipesToRemove);

    // Cek tabrakan dengan pipa
    for (Rectangle pipe : pipes) {
        if (bird.getBoundsInParent().intersects(pipe.getBoundsInParent())) {
            endGame();
            return;
        }
    }
}


    private void spawnPipes() {
        Random random = new Random();
        double gapY = random.nextInt((int) (HEIGHT - PIPE_GAP));

        // Pipa atas
        Rectangle topPipe = new Rectangle(PIPE_WIDTH, gapY, Color.GREEN);
        topPipe.setX(WIDTH);
        topPipe.setUserData("not_scored"); // Menandai pipa belum dilewati

        // Pipa bawah
        Rectangle bottomPipe = new Rectangle(PIPE_WIDTH, HEIGHT - gapY - PIPE_GAP, Color.GREEN);
        bottomPipe.setX(WIDTH);
        bottomPipe.setY(gapY + PIPE_GAP);
        bottomPipe.setUserData("not_scored"); // Menandai pipa belum dilewati

        pipes.add(topPipe);
        pipes.add(bottomPipe);
        pane.getChildren().addAll(topPipe, bottomPipe);
    }

    private void endGame() {
        gameLoop.stop();
        pipeSpawner.stop();
        gameStarted = false;

        // Nampilin teks game over dan restart
        gameOverText.setText("Game Over! Final Score: " + score);
        restartText.setText("Press TAB to Restart");
    }

    private void restartGame() {
        initializeGameElements();
    }
}

