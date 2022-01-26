# Juego-Snake
Codigo del juego del Snake en Netbeans
clase entidad:
ublic class Entidad {
    private int x,y,size;
    
    public Entidad(int size){
    this.size=size;
    }
    public int getX() {
    return x;
    }       
     public int getY() {
    return y;
    }   
     public void setX(int x){
     this.x=x;
     }
    public void setY(int y){
     this.y=y;
     } 
    public void setPosition(int x, int y){
    this.x=x;
    this.y=y;
    }
    public void move(int dx, int dy){
    x+=dx;
    y+=dy;
   
    }
    public Rectangle getBounds(){
    return  new Rectangle(x,y,size,size);
    }
    public boolean isColision(Entidad o){
    if(o==this){
    return false;
    }
    return getBounds().intersects(o.getBounds());
    }
    public void render (Graphics2D g2d){
    g2d.fillRect(x+1, y+1, size-2, size-2);
    }
    
}
Clase panel:
public class Panel extends JPanel implements Runnable, KeyListener {

    public static final int WIDTH = 400;
    public static final int HEIGHT = 400;

    private Graphics2D g2d;
    private BufferedImage image;

    private Thread thread;
    private boolean running;
    private long targetTime;

    private final int SIZE = 10;
    Entidad head, apple;
    ArrayList<Entidad> snake;
    private int score;
    private int level;
    private boolean gameOver;

    private int dx, dy;

    private boolean up, down, rigth, left, start;

    public Panel() {
        setPreferredSize(new Dimension(WIDTH, HEIGHT));
        setFocusable(true);
        requestFocus();
        addKeyListener(this);

    }

    @Override
    public void addNotify() {
        super.addNotify();
        thread = new Thread(this);
        thread.start();
    }

    private void setFPS(int fps) {
        targetTime = 1000 / fps;
    }

    @Override
    public void keyPressed(KeyEvent e) {
        int k = e.getKeyCode();
        if (k == KeyEvent.VK_UP) {
            up = true;
        }
        if (k == KeyEvent.VK_DOWN) {
            down = true;
        }
        if (k == KeyEvent.VK_LEFT) {
            left = true;
        }
        if (k == KeyEvent.VK_RIGHT) {
            rigth = true;
        }
        if (k == KeyEvent.VK_ENTER) {
            start = true;
        }
    }

    @Override
    public void keyReleased(KeyEvent e) {
        int k = e.getKeyCode();
        if (k == KeyEvent.VK_UP) {
            up = false;
        }
        if (k == KeyEvent.VK_DOWN) {
            down = false;
        }
        if (k == KeyEvent.VK_LEFT) {
            left = false;
        }
        if (k == KeyEvent.VK_RIGHT) {
            rigth = false;
        }
        if (k == KeyEvent.VK_ENTER) {
            start = false;
        }
    }

    @Override
    public void keyTyped(KeyEvent e) {
    }

    @Override
    public void run() {
        if (running) {
            return;
        }
        init();
        long startTime;
        long elapse;
        long wait;

        while (running) {
            startTime = System.nanoTime();

            update();
            requestRender();

            elapse = System.nanoTime() - startTime;
            wait = targetTime - elapse / 1000000000; // velocidad de la serpiente
            if (wait > 0) {
                try {
                    Thread.sleep(wait);
                } catch (Exception e) {
                    e.printStackTrace();
                }

            }

        }
    }

    private void init() {
        image = new BufferedImage(WIDTH, HEIGHT, BufferedImage.TYPE_INT_ARGB);
        g2d = image.createGraphics();
        running = true;
        setUplevel();

        setFPS(level * 10);
    }

    private void setUplevel() {
        snake = new ArrayList<Entidad>();
        head = new Entidad(SIZE);
        head.setPosition(WIDTH / 2, HEIGHT / 2);
        snake.add(head);

        for (int i = 1; i < 3; i++) {
            Entidad e = new Entidad(SIZE);
            e.setPosition(head.getX() + (i * SIZE), head.getY());
            snake.add(e);

        }
        apple = new Entidad(SIZE);
        setApple();
        score = 0;
        gameOver = false;
        level = 1;
        dx = dy = 0;
    }

    public void setApple() {
        int x = (int) (Math.random() * (WIDTH - SIZE));
        int y = (int) (Math.random() * (HEIGHT - SIZE));
        x = x - (x % SIZE);
        y = y - (y % SIZE);
        apple.setPosition(x, y);
    }

    private void requestRender() {
        render(g2d);
        Graphics g = getGraphics();
        g.drawImage(image, 0, 0, null);
        g.dispose();
    }

    private void update() {
        if (gameOver) {
            if (start) {
                setUplevel();
            }
            return;
        }
        if (up && dy == 0) {
            dy = -SIZE;
            dx = 0;
        }
        if (down && dy == 0) {
            dy = SIZE;
            dx = 0;
        }
        if (left && dx == 0) {
            dy = 0;
            dx = -SIZE;
        }
        if (rigth && dx == 0 && dy != 0) {
            dy = 0;
            dx = SIZE;
        }
        if (dx != 0 || dy != 0) {
            for (int i = snake.size() - 1; i > 0; i--) {
                snake.get(i).setPosition(snake.get(i - 1).getX(), snake.get(i - 1).getY());
            }
            head.move(dx, dy);
        }
        for (Entidad e : snake) {
            if (e.isColision(head)) {
                gameOver = true;
                break;
            }
        }
        if (apple.isColision(head)) {
            score++;
            setApple();

            Entidad e = new Entidad(SIZE);
            e.setPosition(-100, -100);
            snake.add(e);
            if (score % 10 == 0) {
                level++;
                if (level > 10) {
                    level = 10;

                    setFPS(level - 10);
                }
            }
        }
        if (head.getX() < 0) {
            head.setX(WIDTH);
        }
        if (head.getY() < 0) {
            head.setY(HEIGHT);
        }
        if (head.getX() > WIDTH) {
            head.setX(0);
        }
        if (head.getY() > HEIGHT) {
            head.setY(0);
        }
    }

    public void render(Graphics2D g2d) {
        g2d.clearRect(0, 0, WIDTH, HEIGHT);

        g2d.setColor(Color.GREEN);
        for (Entidad e : snake) {
            e.render(g2d);
        }
        g2d.setColor(Color.red);
        apple.render(g2d);
        if (gameOver) {
            g2d.drawString("GAMEOVER", 150, 200);
        }
        g2d.setColor(Color.WHITE);
        g2d.drawString("SCORE : " + score + "               Level : " + level, 10, 10);

        if (dx == 0 && dy == 0) {
            g2d.drawString("READY", 150, 200);
        }
    }

}
clase Snake:
public class Snake {

    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        JFrame frame=new JFrame("SERPIENTE");
        frame.setContentPane(new Panel());
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setResizable(false);
        frame.pack();
        
        
        frame.setPreferredSize(new Dimension(Panel.WIDTH,Panel.HEIGHT));
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
    
}
