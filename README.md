/**
 * 파일명: DemonSlayerGame.java
 * 설명: 자바 콘솔 기반의 간단한 턴제 귀멸의 칼날 전투 시뮬레이션
 */
import java.util.Scanner;

// --- 1. 호흡 기술 정의 클래스 (Technique) ---
class Technique {
    public String name;
    public int damage;
    public int cost; // 소모되는 집중력(Focus) 게이지

    public Technique(String name, int damage, int cost) {
        this.name = name;
        this.damage = damage;
        this.cost = cost;
    }
}

// --- 2. 기본 캐릭터 추상 클래스 (Character) ---
abstract class Character {
    protected String name;
    protected int health;
    protected int maxHealth;
    protected int focusGauge; // 호흡/혈귀술 사용 게이지
    protected int power; // 기본 공격력

    public Character(String name, int health, int power) {
        this.name = name;
        this.health = health;
        this.maxHealth = health;
        this.power = power;
        this.focusGauge = 10; // 초기 집중력
    }

    // 상태 출력 메소드
    public void displayStatus() {
        System.out.println("[" + name + "] HP: " + health + "/" + maxHealth + " | 집중력: " + focusGauge);
    }
    
    // 공격받는 메소드
    public void takeDamage(int damage) {
        this.health -= damage;
        if (this.health < 0) {
            this.health = 0;
        }
        System.out.println(name + "가 " + damage + "의 피해를 입었습니다.");
    }
    
    // 생존 여부 확인
    public boolean isAlive() {
        return health > 0;
    }

    // 추상 메소드 (자식 클래스에서 반드시 구현)
    public abstract void turn(Character target, Scanner scanner);
}

// --- 3. 귀살대원 클래스 (DemonSlayer) ---
class DemonSlayer extends Character {
    private Technique[] techniques;

    public DemonSlayer(String name, int health, int power, Technique[] techniques) {
        super(name, health, power);
        this.techniques = techniques;
    }

    // 호흡 기술 사용 메소드
    public void useBreathingTechnique(Technique technique, Character target) {
        if (focusGauge >= technique.cost) {
            System.out.println("🔥 " + name + "가 [" + technique.name + "]을(를) 사용합니다!");
            target.takeDamage(technique.damage);
            focusGauge -= technique.cost;
        } else {
            System.out.println("💧 집중력(" + focusGauge + ")이 부족하여 기술을 사용할 수 없습니다! (필요 집중력: " + technique.cost + ")");
        }
    }

    @Override
    public void turn(Character target, Scanner scanner) {
        System.out.println("\n===== " + name + "의 턴 =====");
        displayStatus();
        
        System.out.println("행동을 선택하세요: (1) 기본 공격 (2) 호흡 기술");
        int action = scanner.nextInt();

        if (action == 1) {
            System.out.println(name + "가 일륜도로 기본 공격을 합니다.");
            target.takeDamage(power);
        } else if (action == 2) {
            System.out.println("사용할 호흡 기술을 선택하세요:");
            for (int i = 0; i < techniques.length; i++) {
                System.out.println("(" + (i + 1) + ") " + techniques[i].name + " (피해: " + techniques[i].damage + ", 집중력 소모: " + techniques[i].cost + ")");
            }
            int techChoice = scanner.nextInt() - 1;
            
            if (techChoice >= 0 && techChoice < techniques.length) {
                useBreathingTechnique(techniques[techChoice], target);
            } else {
                System.out.println("잘못된 선택입니다. 기본 공격으로 전환합니다.");
                target.takeDamage(power);
            }
        }
        focusGauge = Math.min(focusGauge + 1, 10); // 턴 종료 시 집중력 회복 (최대 10)
    }
}

// --- 4. 혈귀 클래스 (Demon) ---
class Demon extends Character {
    private String bloodDemonArt;
    private int artDamage;
    private int artCost;

    public Demon(String name, int health, int power, String art, int artDamage, int artCost) {
        super(name, health, power);
        this.bloodDemonArt = art;
        this.artDamage = artDamage;
        this.artCost = artCost;
    }
    
    // 혈귀술 사용
    public void useBloodDemonArt(Character target) {
        if (focusGauge >= artCost) {
            System.out.println("🩸 " + name + "가 혈귀술 [" + bloodDemonArt + "]을(를) 사용합니다!");
            target.takeDamage(artDamage);
            focusGauge -= artCost;
        } else {
            System.out.println("⚡ " + name + "가 집중력 부족으로 기본 공격을 합니다.");
            target.takeDamage(power);
        }
    }

    @Override
    public void turn(Character target, Scanner scanner) {
        System.out.println("\n===== " + name + "의 턴 =====");
        displayStatus();

        // 단순화된 AI: 집중력이 되면 혈귀술 사용, 아니면 기본 공격
        if (focusGauge >= artCost) {
            useBloodDemonArt(target);
        } else {
            System.out.println(name + "가 날카로운 손톱으로 기본 공격을 합니다.");
            target.takeDamage(power);
        }
        focusGauge = Math.min(focusGauge + 2, 10); // 혈귀는 집중력 회복 속도가 빠름
    }
}

// --- 5. 메인 실행 클래스 (DemonSlayerGame) ---
public class DemonSlayerGame {
    
    public static void main(String[] args) {
        System.out.println("=============================================");
        System.out.println("     귀멸의 칼날 - 콘솔 턴제 전투 시작!     ");
        System.out.println("=============================================");

        // 1. 캐릭터 및 기술 정의
        Technique[] tanjiroTechs = new Technique[]{
            new Technique("물의 호흡 제1형: 수면베기", 15, 3),
            new Technique("물의 호흡 제3형: 유유", 25, 5),
            new Technique("물의 호흡 제10형: 생생유전", 40, 8)
        };
        
        DemonSlayer tanjiro = new DemonSlayer("카마도 탄지로", 150, 5, tanjiroTechs);
        
        Demon lowerMoon = new Demon("하현의 5: 루이", 200, 10, "잔혹한 실", 35, 6);
        
        Scanner scanner = new Scanner(System.in);
        
        // 2. 턴제 전투 루프
        while (tanjiro.isAlive() && lowerMoon.isAlive()) {
            
            // 귀살대원 턴
            tanjiro.turn(lowerMoon, scanner);
            if (!lowerMoon.isAlive()) break; // 혈귀 처치 시 전투 종료
            
            // 혈귀 턴
            lowerMoon.turn(tanjiro, scanner);
        }
        
        // 3. 전투 결과 출력
        System.out.println("\n=============================================");
        if (tanjiro.isAlive()) {
            System.out.println("🌟 탄지로의 승리! 하현의 혈귀를 처치했습니다!");
        } else {
            System.out.println("❌ 전투 패배... 다음을 기약해야 합니다.");
        }
        System.out.println("=============================================");
        
        scanner.close();
    }
}
