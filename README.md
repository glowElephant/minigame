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

## 2. 인앱결제 대한 에러처리

### 설명

인앱결제에 프로세스에대해 확인 후 해당 결제에대한 에러처리를 추가

### 처리부분

<pre><code>
 /// <summary>
        /// https://docs.unity3d.com/ScriptReference/Purchasing.PurchaseFailureReason.html
        /// 유니티 IAPManager 구매 에러 원인들
        /// </summary>
        /// <param name="ex"></param>
        public void HandlePurchaseError(PurchaseException ex)
		{
			switch (ex.FailReason)
			{
			case PurchaseFailureReason.DuplicateTransaction:
				{
					// 스토어 결재 완료. 서버 처리 안됨.
					ConfirmPendingPurchase(ex.PurchasedProduct);
					DlgMessageBox.Show("ERR_PLEASE_REPORT");
				}
				break;

			case PurchaseFailureReason.SignatureInvalid:
				// 구매 영수증의 서명 확인 실패
				DlgMessageBox.Show("ERR_SIGNATURE_INVALID");
				break;

			case PurchaseFailureReason.UserCancelled:
				// 사용자 구매 취소
				DlgMessageBox.Show("ERR_USER_CANCLLED");
				break;

			case PurchaseFailureReason.PaymentDeclined:
				// 사용자가 지불하는데에 문제가 있음
				DlgMessageBox.Show("ERROR_IAP_INIT");
				break;

			case PurchaseFailureReason.ExistingPurchasePending:
				// 이미 구매가 진행중인 상태
				DlgMessageBox.Show("ERR_EXISTING_PURCHASE_PENDING");
				break;

			default:
				DlgMessageBox.Show("ERR_PLEASE_REPORT");
				break;
			}

			//실패시 서버로 리포트는 무조건 해야할것같다....(관리차원) 유저가 취소했을때 빼고!
			if (ex.FailReason != PurchaseFailureReason.UserCancelled)
			{
				LogEvent.ReportException("IAPManager", ex);
			}
		}
</code></pre>

## 3. 행성연구 컨텐츠 개발

### 설명

행성 연구 컨텐츠라는 게임 컨텐츠를 개발하였다 다른 컨텐츠와는 다르게
해당 컨텐츠는 서버와의 통신에서 json 형태의 string을 주고 받았다

### 프로토콜
<pre><code>
#region ReqSaveLaboratory
	public static IPromise<ReqSaveLaboratoryResponse> ReqSaveLaboratory(PlanetResearchContext context)
	{
		var json = JsonConvert.SerializeObject(context, Formatting.Indented);
		var pf = Profiler.Start("ReqSaveLaboratory");
		Promise<ReqSaveLaboratoryResponse> promise = new Promise<ReqSaveLaboratoryResponse>();
		new LogEventRequest()
			.SetEventKey("ReqSaveLaboratory")
			.SetEventAttribute("Args", new GSRequestData()
			.AddNumber("LaboratoryTid", context.TID)
			.AddJSONStringAsObject("Data", json))
			.SetMaxResponseTimeInMillis(kMaxResponseTime)
			.Send(response =>
			{
				pf.StopAndReport();
				if (response.HasErrors)
				{
					promise.Reject(new GameSparkException(response.Errors));
				}
				else
				{
					promise.Resolve(new ReqSaveLaboratoryResponse(response));
				}
			});

		return promise;
	}

	public class ReqSaveLaboratoryResponse // : CurrencyResponse
	{
		public int? Version { get; private set; }

		public ReqSaveLaboratoryResponse(LogEventResponse response) //: base(response)
		{
			var result = response.ScriptData.GetGSData("Result");
			Version = result.GetInt("Version");
		}
	}

	#endregion ReqSaveLaboratory
</code></pre>
