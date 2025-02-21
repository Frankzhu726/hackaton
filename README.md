# hackaton
git add BoidsSimulation.java
git commit -m "Add Boids Simulation"
git push origin main

import javax.swing.*;
import java.awt.*;
import java.util.ArrayList;
import java.util.Random;

public class BoidsSimulation extends JPanel {
    private static final int WIDTH = 800;
    private static final int HEIGHT = 600;
    private static final int NUM_BOIDS = 100;
    private static final int BOID_SIZE = 5;
    private static final double MAX_SPEED = 4.0;
    private static final double VIEW_RADIUS = 50.0;

    private ArrayList<Boid> boids;

    public BoidsSimulation() {
        boids = new ArrayList<>();
        Random random = new Random();

        // Initialize boids with random positions and velocities
        for (int i = 0; i < NUM_BOIDS; i++) {
            double x = random.nextDouble() * WIDTH;
            double y = random.nextDouble() * HEIGHT;
            double vx = random.nextDouble() * MAX_SPEED * 2 - MAX_SPEED;
            double vy = random.nextDouble() * MAX_SPEED * 2 - MAX_SPEED;
            boids.add(new Boid(x, y, vx, vy));
        }
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.BLACK);
        g.fillRect(0, 0, WIDTH, HEIGHT);

        g.setColor(Color.WHITE);
        for (Boid boid : boids) {
            g.fillOval((int) boid.x, (int) boid.y, BOID_SIZE, BOID_SIZE);
        }
    }

    public void update() {
        for (Boid boid : boids) {
            boid.update(boids);
        }
        repaint();
    }

    public static void main(String[] args) {
        JFrame frame = new JFrame("Boids Simulation");
        BoidsSimulation simulation = new BoidsSimulation();

        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(WIDTH, HEIGHT);
        frame.add(simulation);
        frame.setVisible(true);

        // Animation loop
        new Timer(16, e -> simulation.update()).start();
    }

    static class Boid {
        double x, y;
        double vx, vy;

        public Boid(double x, double y, double vx, double vy) {
            this.x = x;
            this.y = y;
            this.vx = vx;
            this.vy = vy;
        }

        public void update(ArrayList<Boid> boids) {
            double[] separation = calculateSeparation(boids);
            double[] alignment = calculateAlignment(boids);
            double[] cohesion = calculateCohesion(boids);

            // Update velocity based on rules
            vx += separation[0] + alignment[0] + cohesion[0];
            vy += separation[1] + alignment[1] + cohesion[1];

            // Limit speed
            double speed = Math.sqrt(vx * vx + vy * vy);
            if (speed > MAX_SPEED) {
                vx = (vx / speed) * MAX_SPEED;
                vy = (vy / speed) * MAX_SPEED;
            }

            // Update position
            x += vx;
            y += vy;

            // Wrap around screen edges
            if (x < 0) x += WIDTH;
            if (x > WIDTH) x -= WIDTH;
            if (y < 0) y += HEIGHT;
            if (y > HEIGHT) y -= HEIGHT;
        }

        private double[] calculateSeparation(ArrayList<Boid> boids) {
            double steerX = 0;
            double steerY = 0;
            int count = 0;

            for (Boid other : boids) {
                double distance = Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
                if (other != this && distance < VIEW_RADIUS / 2) {
                    steerX += x - other.x;
                    steerY += y - other.y;
                    count++;
                }
            }

            if (count > 0) {
                steerX /= count;
                steerY /= count;
            }

            return new double[]{steerX * 0.05, steerY * 0.05};
        }

        private double[] calculateAlignment(ArrayList<Boid> boids) {
            double avgVx = 0;
            double avgVy = 0;
            int count = 0;

            for (Boid other : boids) {
                double distance = Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
                if (other != this && distance < VIEW_RADIUS) {
                    avgVx += other.vx;
                    avgVy += other.vy;
                    count++;
                }
            }

            if (count > 0) {
                avgVx /= count;
                avgVy /= count;
            }

            return new double[]{(avgVx - vx) * 0.05, (avgVy - vy) * 0.05};
        }

        private double[] calculateCohesion(ArrayList<Boid> boids) {
            double centerX = 0;
            double centerY = 0;
            int count = 0;

            for (Boid other : boids) {
                double distance = Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
                if (other != this && distance < VIEW_RADIUS) {
                    centerX += other.x;
                    centerY += other.y;
                    count++;
                }
            }

            if (count > 0) {
                centerX /= count;
                centerY /= count;
            }

            return new double[]{(centerX - x) * 0.01, (centerY - y) * 0.01};
        }
    }
}
