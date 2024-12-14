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

abstract class GameObject {
    protected double x, y;
    protected Rectangle shape;

    public abstract void update();  

    public double getX() {
        return x;
    }
    public double getY() {
        return y;
    }
    public Rectangle getShape() {
        return shape;
    }
    public void setX(double x) {
        this.x = x;
    }
    public void setY(double y) {
        this.y = y;
    }
}

class Bird extends GameObject {
    private double velocity;
    private static final double FLAP_STRENGTH = -10;
    private static final double GRAVITY = 0.5;

    public Bird(double initialX, double initialY) {
        this.x = initialX;
        this.y = initialY;
        this.velocity = 0;
        this.shape = new Rectangle(20, 20, Color.YELLOW);
        this.shape.setX(x);
        this.shape.setY(y);
    }

    @Override
    public void update() {
        velocity += GRAVITY;
        y += velocity;
        shape.setY(y);
    }

    public void flap() {
        velocity = FLAP_STRENGTH;
    }
}

class Pipe extends GameObject {
    private static final double PIPE_WIDTH = 50;
    private static final double PIPE_GAP = 150;

    public Pipe(double x, double gapY) {
        this.x = x;
        this.y = gapY;
        this.shape = new Rectangle(PIPE_WIDTH, gapY, Color.GREEN);
        this.shape.setX(x);
    }

    @Override
    public void update() {
        x -= 3;
        shape.setX(x);
    }
}

public class FlappyBirdGame extends Application {
    private static final double WIDTH = 600;
    private static final double HEIGHT = 400;
    private static final double BIRD_SIZE = 20;

    private Bird bird;
    private List<Pipe> pipes;
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

        initializeGameElements();

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
        bird = new Bird(100, HEIGHT/2);
        pipes = new ArrayList<>();
        score = 0;
        gameStarted = false;

        // Reset tampilan
        pane.getChildren().clear();

        // Teks skor
        scoreText = new Text(10, 20, "Score: 0");
        scoreText.setFill(Color.BLACK);

        // Teks "Game Over"
        gameOverText = new Text(WIDTH / 2 - 100, HEIGHT / 2, "");
        gameOverText.setFill(Color.BLACK);

        // Teks "Press TAB to Restart"
        restartText = new Text(WIDTH / 2 - 100, HEIGHT / 2 + 30, "");
        restartText.setFill(Color.BLACK);

        startText = new Text(WIDTH / 2 - 150, HEIGHT / 2 - 30, "Tekan Spasi untuk Memulai dan Terbang");
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

        // Loop buat pipa
        pipeSpawner = new Timeline(new KeyFrame(Duration.seconds(2), e -> spawnPipes()));
        pipeSpawner.setCycleCount(Timeline.INDEFINITE);
        pipeSpawner.play();
    }

    private void gameUpdate() {
    bird.update();
    
    // Cek burung keluar layar
    if (bird.getY > HEIGHT - BIRD_SIZE || bird.getY < 0) {
        endGame();
        return;
    }

    // Update posisi pipa
    List<Pipe> pipesToRemove = new ArrayList<>();
    for (Pipe pipe : pipes) {
        pipe.update();

        // Tambah skor jika burung melewati pipa
        if (pipe.getX() + 50 < bird.getX() && pipe.getShape().getUserData() == null) {
            score++;
            pipe.getShape().setUserData("scored");
            scoreText.setText("Score: " + score);
        }

        // Hapus pipa jika keluar layar
        if (pipe.getX() + 50 < 0) {
            pipesToRemove.add(pipe);
            }
        }

    pipes.removeAll(pipesToRemove);
    pane.getChildren().removeAll(pipesToRemove);

    // Cek tabrakan dengan pipa
    for (Pipe pipe : pipes) {
        if (bird.getShape().getBoundsInParent().intersects(pipe.getShape().pipe.getBoundsInParent())) {
            endGame();
            return;
        }
    }
}

    private void spawnPipes() {
        Random random = new Random();
        double gapY = random.nextInt((int) (HEIGHT - 150));

        Pipe topPipe = new Pipe(WIDTH, gapY);
        Pipe bottomPipe = new Pipe(WIDTH, HEIGHT - gapY - 150);
        bottomPipe.getShape().setY(gapY + 150);

        pipes.add(topPipe);
        pipes.add(bottomPipe);
        pane.getChildren().addAll(topPipe.getShape(), bottomPipe.getShape());
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
        try{
            initializeGameElements();
        } catch(Exception e){
            System.out.println("Game gagal dimuat " + e.getMessage());
        }
    }
}
