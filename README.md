# portfolio
---------------------------------------
## 1. 미니게임 가위바위보

![Alt Text](./resources/minigamecombat.gif)

### 설명

환형 링크드 리스트를 활용하여 구현

### 노드 데이터 구조
<pre><code>
public class RPSNode
{
	public RPSGame Elem { get; set; }	// 가위인지 바위인지 보인지를 결정
	
	public RPSNode Next { get; set; }	// 다음 노드

	public RPSNode Previous { get; set; }	// 이전 노드
}
</code></pre>

### 리스트 구조

<pre><code>
/// <summary>
/// 가위바위보 판별을 위한 데이터 셋팅
/// </summary>
public class RPSNodeList
{
	public RPSNode RockNode { get; private set; }
	public RPSNode PaperNode { get; private set; }
	public RPSNode ScissorNode { get; private set; }

	/// <summary>
	/// 환형 리스트로 구조를 만들어 다음과 이전을 참조하도록 한다
	/// </summary>
	public List<RPSNode> RPSList { get; private set; }

	public RPSNodeList()
	{
		RockNode = new RPSNode();
		PaperNode = new RPSNode();
		ScissorNode = new RPSNode();

		RockNode.Elem = RPSGame.Rock;
		PaperNode.Elem = RPSGame.Paper;
		ScissorNode.Elem = RPSGame.Scissor;

		RockNode.Next = PaperNode;
		PaperNode.Next = ScissorNode;
		ScissorNode.Next = RockNode;

		RockNode.Previous = ScissorNode;
		PaperNode.Previous = RockNode;
		ScissorNode.Previous = PaperNode;

		RPSList = new List<RPSNode>();

		RPSList.Add(RockNode);
		RPSList.Add(PaperNode);
		RPSList.Add(ScissorNode);
	}
	
	/// <summary>
	/// elem을 통해 몇번째에 해당 elem이 있는지 확인
	/// </summary>
	/// <param name="elem">가위 바위 보</param>
	/// <returns>해당 인덱스</returns>
	public int? SearchingEqualIndex(RPSGame elem)
	{
		for (int i = 0; i < RPSList.Count; i++)
		{
			if (elem == RPSList[i].Elem)
			{
				return i;
			}
		}
		return null;
	}
}
</code></pre>

## 2. 미니게임 가위바위보

