
import java.awt.Font;
import java.awt.Graphics;

import javax.swing.JFrame;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;
public class PongGame extends JFrame {
    private static final int WIDTH = 600;
    private static final int HEIGHT = 400;
    private static final int PADDLE_WIDTH = 20;
    private static final int PADDLE_HEIGHT = 80;
    private static final int BALL_SIZE = 20;
    private static final int MAX_CHANCES = 5;

    private int paddle1Y = HEIGHT / 2 - PADDLE_HEIGHT / 2;
    private int paddle2Y = HEIGHT / 2 - PADDLE_HEIGHT / 2;
    private int ballX = WIDTH / 2 - BALL_SIZE / 2;
    private int ballY = HEIGHT / 2 - BALL_SIZE / 2;
    private int ballXSpeed = 5;
    private int ballYSpeed = 2;

    private boolean upPressed = false;
    private boolean downPressed = false;

    private int score1 = 0;
    private int score2 = 0;

    private int chancesMissedPlayer1 = 0;
    private int chancesMissedPlayer2 = 0;

    public PongGame() {
        setTitle("Pong Game");
        setSize(WIDTH, HEIGHT);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setResizable(false);

        Timer timer = new Timer(10, new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                if (chancesMissedPlayer1 < MAX_CHANCES && chancesMissedPlayer2 < MAX_CHANCES) {
                    update();
                    repaint();
                } else {
                    endGame();
                }
            }
        });
        timer.start();

        addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                handleKeyPress(e);
            }

            @Override
            public void keyReleased(KeyEvent e) {
                handleKeyRelease(e);
            }
        });

        setFocusable(true);
    }

    private void handleKeyPress(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_UP) {
            upPressed = true;
        } else if (e.getKeyCode() == KeyEvent.VK_DOWN) {
            downPressed = true;
        }
    }

    private void handleKeyRelease(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_UP) {
            upPressed = false;
        } else if (e.getKeyCode() == KeyEvent.VK_DOWN) {
            downPressed = false;
        }
    }

    private void update() {
        // Update paddles
        if (upPressed && paddle1Y > 0) {
            paddle1Y -= 5;
        }
        if (downPressed && paddle1Y < HEIGHT - PADDLE_HEIGHT) {
            paddle1Y += 5;
        }

        // Update ball
        ballX += ballXSpeed;
        ballY += ballYSpeed;

        // Check collisions with paddles
        if (ballX <= PADDLE_WIDTH && ballY + BALL_SIZE >= paddle1Y && ballY <= paddle1Y + PADDLE_HEIGHT) {
            ballXSpeed = -ballXSpeed;
            score1++;
        }

        if (ballX + BALL_SIZE >= WIDTH - PADDLE_WIDTH && ballY + BALL_SIZE >= paddle2Y
                && ballY <= paddle2Y + PADDLE_HEIGHT) {
            ballXSpeed = -ballXSpeed;
            score2++;
        }

        // Check collisions with walls
        if (ballY <= 0 || ballY + BALL_SIZE >= HEIGHT) {
            ballYSpeed = -ballYSpeed;
        }

        // Check if the ball went out of bounds
        if (ballX < 0) {
            chancesMissedPlayer1++;
            resetBall();
        } else if (ballX + BALL_SIZE > WIDTH) {
            chancesMissedPlayer2++;
            resetBall();
        }

        // AI for the second paddle
        if (paddle2Y + PADDLE_HEIGHT / 2 < ballY + BALL_SIZE / 2 && paddle2Y < HEIGHT - PADDLE_HEIGHT) {
            paddle2Y += 2;
        } else if (paddle2Y + PADDLE_HEIGHT / 2 > ballY + BALL_SIZE / 2 && paddle2Y > 0) {
            paddle2Y -= 2;
        }
    }

    private void resetBall() {
        ballX = WIDTH / 2 - BALL_SIZE / 2;
        ballY = HEIGHT / 2 - BALL_SIZE / 2;
        ballXSpeed = -ballXSpeed;
    }

    private void endGame() {
        String winner;
        if (score1 > score2) {
            winner = "Player 1";
        } else if (score2 > score1) {
            winner = "Player 2";
        } else {
            winner = "It's a Tie!";
        }

        JOptionPane.showMessageDialog(null, "Game Over!\nFinal Scores:\nPlayer 1: " + score1 + "\nPlayer 2: " + score2
                + "\nWinner: " + winner);
        System.exit(0);
    }

    @Override
    public void paint(Graphics g) {
        super.paint(g);

        // Draw paddles
        g.fillRect(0, paddle1Y, PADDLE_WIDTH, PADDLE_HEIGHT);
        g.fillRect(WIDTH - PADDLE_WIDTH, paddle2Y, PADDLE_WIDTH, PADDLE_HEIGHT);

        // Draw ball
        g.fillOval(ballX, ballY, BALL_SIZE, BALL_SIZE);

        // Draw scores
        g.setFont(new Font("Arial", Font.BOLD, 20));
        g.drawString("Score: " + score1 + " - " + score2, WIDTH / 2 - 60, 30);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new PongGame().setVisible(true));
    }
}
