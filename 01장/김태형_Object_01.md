# 1장 객체, 설계 ch1

## 이론 vs 실무?

오브젝트 서적의 ch1 객체, 설계에서는 프로그래밍을 공부할 때, 이론과 실무중에서 우선순위를 어느 부분에 둬야 하는지를 먼저 의논하고 시작한다.

공학의 어느 분야를 막론하고, 이론을 정립할 수 없는 초기에는 실무가 먼저 급속도로 발전을 하게된다. 실무가 발전을 우선적으로 해야지 실용성의 입증이 가능하고 실용성이 입증이 되어야 이론이 정립 되기 때문이다.

그렇기 때문에 해당 분야가 성숙해지는 시점에서 탄탄하게 쌓인 실무에 의해서 이론이 발전되고, 그 시점에서 이론이 실무를 추월하게 되는것이다.

로버트 L. 글래스는 현재 소프트웨어 분야는 아직 걸음마 단계이기 때문에 이론보다 실무가 더 앞서 있으며 실무가 더 중요하다고 보고 있다.

이 책에서는 해당 관점에 대해서 적극 동의하고 있다. 책의 모든 서술 방식이 실무적인 관점 즉, 코드의 흐름과 요구사항을 먼저 설명한 뒤, 이에 따른 이론들을 서술해주는 방식을 취하고 있다.

## 티켓 판매 어플리케이션

실무의 관점에서 출발점을 잡기 위해서 먼저 티켓 판매 어플리케이션을 예시로 들어보겠다.

티켓 판매 어플리케이션은

1. 관람객이 공연을 관람하기 위해서는 티켓이 필요하다.
2. 티켓은 매표소로부터 현금으로 구매할 수 있고,
3. 회사로부터 받은 초대장을 사용해서 교환받을 수 있다.

이러한 요구사항을 구현하기 위한 Theater 클래스의 enter 메서드를 정의하자.

요구사항을 진행하는 과정에서 수행되어야 하는 동작은 다음과 같다.

1. 티켓을 획득하면, Bag내부에 ticket필드가 해당 티켓으로 set 되어야 한다.
2. 티켓을 구매하면, Bag내부에 Amount가 줄어야한다.
3. 티켓을 구매하면, ticketOffice가 가지고 있는 Amount(금액)이 증가해야 한다.

요구사항을 충족하기 위한 흐름은 다음과 같다.

- audience.getBag().hasInvitation()을 통해 티켓을 가지고 있는지 확인한다.
    
    있다면
    
    - ticketOffice.getTicket()으로 티켓을 발급한다.
    - audience.getBag().setTicket(ticket)으로 ticket을 소지 설정한다.
    
    없다면
    
    - ticketOffice.getTicket()으로 티켓을 발급한다.
    - audience.getBag().minusAmount(ticket.getFee())로 관람객의 소지금액을 줄인다.
    - ticketOffice.plusAmount(ticket.getFee())로 매표소의 매출을 늘린다.
    - audience.getBag().setTicket(ticket)으로 ticket을 소지 설정한다.

> 티켓 판매 어플리케이션 코드
> 

초대장이라는 개념을 구현한 Invitation

- 소스코드
    
    ```java
    public class Invitation{
    	private LocalDateTime when;
    }
    ```
    

사람들이 가지고 있는 티켓 (Ticket)

- 소스코드
    
    ```java
    public class Ticket{
    	private Long fee;
    	public Long getFee(){
    		return fee;
    	}
    }
    ```
    

사람들이 가지고 있는 소지품을 보관하는 용도의 가방

- 소스코드
    
    ```java
    public class Bag{
    	private Long amount;
    	private Invitation invitation;
    	private Ticket ticket;
    
    	public boolean hasInvitation(){
    		return invitation != null;
    	}
    
    	public boolean hasTicket(){
    		return ticket != null;
    	}
    
    	public void setTicket(Ticket ticket){
    		this.ticket = ticket;
    	}
    
    	public void minusAmount(Long amount){
    		this.amount -= amount;
    	}
    
    	public void plusAmount(Long amount){
    		this.amount += amount;
    	}
    
    	public Bag(Long amount){
    		this(null, amount);
    	}
    
    	public Bag(Invitation invitation, Long amount){
    		this.invitation = invitation;
    		this.amount = amount;
    	}
    }
    ```
    

관람객을 개념을 구현하는 Audience

- 소스코드
    
    ```java
    public class Audience{
        public Bag bag;
        public Audience(Bag bag){
            this.bag = bag;
        }
        public Bag getBag(){
            return bag;
        }
    }
    ```
    

티켓을 판매하는 매표소

- 소스코드
    
    ```java
    public class TicketOffice{
        private Long amount;
        private List<Ticket> tickets = new ArrayList<>();
        public TicketOffice(Long amount, Ticket ... tickets){
            this.amount=amount;
            this.tickets.addAll(Arrays.asList(tickets));
        }
        public Ticket getTicket(){
            return tickets.remove(0);
        }
        public void minusAmount(Long amount){
            this.amount -= amount;
        }
        public void plusAmount(Long amount){
            this.amount += amount;
        }
    }
    ```
    

초대장을 티켓으로 교환하거나 티켓을 판매하는 역할을 수행하는 판매원

- 소스코드
    
    ```java
    public class TicketSeller{
        private TicketOffice ticketOffice;
        public TicketSeller(TicketOffice ticketOffice){
            this.ticketOffice = ticketOffice;
        }
        
        public TicketOffice getTicketOffice(){
            return ticketOffice;
        }
    }
    ```
    

관람객의 입장을 진행하는 Theater

- 소스코드
    
    ```java
    public class Theater{
        private TicketSeller ticketSeller;
        public Theater(TicketSeller ticketSeller){
            this.ticketSeller = ticketSeller;
        }
        
        public void enter(Audience audience){
            if(audience.getBag().hasInvitation()){
                Ticket ticket = ticketSeller.getTicketOffice().getTicket();
                audience.getBag().setTicket(ticket);
            }else{
                Ticket ticket = ticketSeller.getTicketOffice().getTicket();
                audience.getBag().minusAmount(ticket.getFee());
                ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
                audience.getBag().setTicket(ticket);
            }
        }
    }
    ```
    

위의 코드들의 역할 분배를 살펴보면 다음과 같다. (배경 색으로 설정)

- audience.getBag().hasInvitation()을 통해 티켓을 가지고 있는지 확인한다. (Theater)
    
    있다면 (Theater)
    
    - ticketOffice.getTicket()으로 티켓을 발급한다. (Theater)
    - audience.getBag().setTicket(ticket)으로 ticket을 소지 설정한다. (Theater)
    
    없다면 (Theater)
    
    - ticketOffice.getTicket()으로 티켓을 발급한다. (Theater)
    - audience.getBag().minusAmount(ticket.getFee())로 관람객의 소지금액을 줄인다. (Theater)
    - ticketOffice.plusAmount(ticket.getFee())로 매표소의 매출을 늘린다. (Theater)
    - audience.getBag().setTicket(ticket)으로 ticket을 소지 설정한다. (Theater)

즉, 모든 코드의 로직이 Theater에 들어가 있기 때문에 코드의 수정이 전부 Theater에서 일어나야 하고, 흐름 또한 전부 파악해야지 수정할 부분을 알 수 있다.

로버트마틴의 클린 소프트웨어에서는 소프트웨어 묘듈이 가져야 하는 세 가지 기능에 대해서 설명한다.

요약하자면 다음과 같다.

1. 제대로 동작해야 하고
2. 변경이 용이해야 하며
3. 이해하기 쉬워야 한다.

그러나 위의 코드는 제대로 동작은 하지만 변경하기에도 무리가 있고, 이해하기에도 쉽지 않다.

흐름이 전부 하나의 메서드에 있기 때문에 변경을 하려면 메서드 안의 로직을 살펴야 하고, 역할의 분배가 직관적이지 않기 때문이다.

이것은 객체의 **의존성**과 관련된 문제이다. Theater의 객체가 로직에서 실행되는 모든 객체를 가지고 있기 때문이다. 이 문제를 해결할 방법은 **객체에게 역할을 분배**하는 것이다.

Theater의 코드를 다음과 같이 바꾸고, TicketSeller를 다음과 같이 바꾸면 이 문제를 해결할 수 있다.

- Theater 코드
    
    ```java
    public class Theater{
        private TicketSeller ticketSeller;
        public Theater(TicketSeller ticketSeller){
            this.ticketSeller = ticketSeller;
        }
    
        public void enter(Audience audience){
            ticketSeller.toSell(audience);
        }
    }
    ```
    
- TicketSeller 코드
    
    ```java
    public class TicketSeller{
        private TicketOffice ticketOffice;
        public TicketSeller(TicketOffice ticketOffice){
            this.ticketOffice = ticketOffice;
        }
        
        public void toSell(Audience audience){
            if(audience.getBag().hasInvitation()){
                Ticket ticket = ticketOffice.getTicket();
                audience.getBag().setTicket(ticket);
            }else{
                Ticket ticket = ticketOffice.getTicket();
                audience.getBag().minusAmount(ticket.getFee());
                ticketOffice.plusAmount(ticket.getFee());
                audience.getBag().setTicket(ticket);
            }
        }
    }
    ```
    (Theater)
- audience.getBag().hasInvitation()을 통해 티켓을 가지고 있는지 확인한다. (TicketSeller)
    
    있다면 (TicketSeller)
    
    - ticketOffice.getTicket()으로 티켓을 발급한다. (TicketSeller)
    - audience.getBag().setTicket(ticket)으로 ticket을 소지 설정한다. (TicketSeller)
    
    없다면 (TicketSeller)
    
    - ticketOffice.getTicket()으로 티켓을 발급한다. (TicketSeller)
    - audience.getBag().minusAmount(ticket.getFee())로 관람객의 소지금액을 줄인다. (TicketSeller)
    - ticketOffice.plusAmount(ticket.getFee())로 매표소의 매출을 늘린다. (TicketSeller)
    - audience.getBag().setTicket(ticket)으로 ticket을 소지 설정한다. (TicketSeller)

Theater의 코드만 보았을 때, 우리는 toSell 내부에서 어떤일이 일어나는지는 모르지만, 티켓을 교환하는 로직이 일어난다는것은 알 수 있다. 또한 파는 로직에 대한 수정이 일어났을 때, Theater코드가 아닌, 구매 로직이 구현되어 있는 TicketSeller의 toSell만 수정하면 된다.

이렇게 객체 내부의 세부적인 사항을 감추는 것을 **캡슐화** 라고 하고, 캡슐화의 목적은 변경하기 쉬운 객체를 만드는 것이다.

따라서 수정된 코드는  변경 용이성의 축면에서도 확실히 개선됐다고 말할 수 있다.