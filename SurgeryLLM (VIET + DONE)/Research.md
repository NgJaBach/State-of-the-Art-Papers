---


---

<h1 id="code--docs"><a href="https://github.com/NgJaBach/State-of-the-Art-Papers/tree/main/SurgeryLLM">Code + Docs</a></h1>
<p><img src="https://dl.imgdrop.io/file/aed8b140-8472-4813-922b-7ce35ef93c9e/2025/03/24/41746_2024_1391_Fig1_HTML556e12872fdd591e.webp" alt="surgeryllm"></p>
<h1 id="lựa-chọn-model">Lựa chọn model:</h1>
<p>Trong bài báo, tác giả sử dụng từ model Llama 2 cho đến 3.2, nhưng để cho đơn giản, em sẽ giả sử tác giả sử dụng Llama 3.2.</p>
<p>Để đảm bảo model sử dụng của chúng ta đủ tốt, ta muốn một model đạt MMLU tương đương với Llama 3.2, đồng thời muốn giá của model đó rẻ hơn so với model bài báo sử dụng (vì tự chạy Llama khá là khoai). Claude và GPT thường khá là đắt, do đó ta dễ dàng loại trừ được một số model cấp cao của họ. Ngoài ra thì các model đạt MMLU dưới 80% ta đều bỏ hết.</p>
<p>Tham khảo benchmark MMLU (khả năng xử lý ngôn ngữ) và giá cả, ta có một số models nổi bật như sau:</p>

<table>
<thead>
<tr>
<th>Model</th>
<th>Parameters (B)</th>
<th>Input $/M</th>
<th>Output $/M</th>
<th>MMLU</th>
</tr>
</thead>
<tbody>
<tr>
<td>DeepSeek-V3</td>
<td>671</td>
<td>$0.27</td>
<td>$1.10</td>
<td>88.5%</td>
</tr>
<tr>
<td>Llama 3.2 90B Instruct</td>
<td>90</td>
<td>$0.35</td>
<td>$0.40</td>
<td>86.0%</td>
</tr>
<tr>
<td>Llama 3.3 70B Instruct</td>
<td>70</td>
<td>$0.20</td>
<td>$0.20</td>
<td>86.0%</td>
</tr>
</tbody>
</table><p>Các model trên đều có context-window khá lớn (128k+ tokens) nên phù hợp với bài toán RAG. Do ta sử dụng RAG nên đầu vào thường sẽ nhiều token hơn output, do đó ta thấy model Llama 3.3 70B có vẻ là sự lựa chọn tốt hơn so với Llama 3.2 90B, và đủ tốt nếu so với Deepseek V3. Ta cũng sẽ sử dụng thêm Llama 3.2 3B để phục vụ mục đích test code và cho kết quả nhanh chóng.</p>
<h1 id="code">Code</h1>
<p>Thông tin về prompt và kết quả của tác giả: <a href="https://static-content.springer.com/esm/art%3A10.1038%2Fs41746-024-01391-3/MediaObjects/41746_2024_1391_MOESM1_ESM.pdf">Supplementary Information</a></p>
<p>Do máy em không có card nvidia cũng như khá yếu, em sẽ sử dụng vector store <strong>FAISS</strong> thay cho Chroma, vì nó hỗ trợ xử lý đa luồng CPU cũng như GPU (nếu có).</p>
<p>Em có thử RAG lên web hoặc pdf online của tài liệu, nhưng nó đều bị lỗi 403. Em tìm hiểu một lúc thì thấy có vẻ cái này khá khó giải quyết, vì một số web chủ ý chặn các request từ bot. Do vậy em tải các pdf về thủ công rồi gọi RAG lên các pdf đấy, sử dụng <strong>PyPDFLoader</strong>.</p>
<p>Cách thức thực hiện chủ yếu khá giống page tài liệu nhập môn RAG của LangChain: <a href="https://python.langchain.com/docs/concepts/retrieval/">Retrieval | 🦜️🔗 LangChain</a></p>
<p>Vì <strong>GPT4All Embeddings</strong> rất nặng, nên mỗi lần tạo vector database đều rất chậm, như máy em là mất khoảng 15-20 phút. Mà bản chất của vector_store chỉ là lưu trữ lại các chunk của các văn bản, nên xuyên suốt bài toán sẽ không có thay đổi gì. Vậy em thực hiện lưu trữ vector_store này và gọi nó ra nếu nó đã có sẵn, trừ khi em muốn thay đổi cái database đó. Tham khảo từ tài liệu về FAISS: <a href="https://python.langchain.com/docs/integrations/vectorstores/faiss/#query-by-turning-into-retriever">Faiss | 🦜️🔗 LangChain</a></p>
<h1 id="cải-tiến-chính">Cải tiến chính</h1>
<p>Một thay đổi nữa, đó là search type, thay vì để mặc định là similarity search, thì em để <strong>mmr</strong> (## <a href="https://www.bing.com/ck/a?!&amp;&amp;p=fb671dba8b091350d4c411b518f5dd63bcb644e7d7a4f7de892d8c9954e3c9a7JmltdHM9MTc0MTk5NjgwMA&amp;ptn=3&amp;ver=2&amp;hsh=4&amp;fclid=2f5c8f6a-136b-657f-0786-9ae612b2641b&amp;psq=mmr+search&amp;u=a1aHR0cHM6Ly9tZWRpdW0uY29tL3RlY2gtdGhhdC13b3Jrcy9tYXhpbWFsLW1hcmdpbmFsLXJlbGV2YW5jZS10by1yZXJhbmstcmVzdWx0cy1pbi11bnN1cGVydmlzZWQta2V5cGhyYXNlLWV4dHJhY3Rpb24tMjJkOTUwMTVjN2M1&amp;ntb=1">Maximal Marginal Relevance</a>) search. Dạng này giúp mở rộng phạm vi tìm kiếm hơn, không giới hạn tiêu chí tìm kiếm với các đoạn tương đồng nhất, mà đa dạng kết quả nhất, đánh đổi bằng việc bỏ qua một số đoạn rất tương đồng. Nhưng nếu nhìn vào thực tế thì tài liệu của chúng ta chứa rất nhiều đoạn “thừa”, gây nhiễu, nên việc chỉ xét tương đồng có thể trả về kết quả vô nghĩa.</p>
<p>Ví dụ:</p>
<blockquote>
<p>being considered for coronary revascularization, with the intent to<br>
improve quality of care and align with patients’ interests. Key Words:<br>
AHA Scientific Statements ◼ percutaneous coronary intervention ◼<br>
angioplasty ◼ coronary artery bypass graft surgery ◼ myocardial<br>
infarction ◼ cardiac surgery, stent(s) ◼ angiogram ◼ angiography ◼<br>
percutaneous transluminal coronary angioplasty ◼ coronary<br>
atherosclerosis ◼ saphenous vein graft ◼ internal mammary artery graft<br>
◼ internal thoracic artery graft ◼ arterial graft ◼ post-bypass ◼<br>
non–ST-segment–elevated myocardial infarction ◼ vein graft lesions ◼<br>
myocardial revascularization ◼ multivessel PCI ◼ left ventricular<br>
dysfunction Clinical Statements and Guidelines Downloaded from<br>
<a href="http://ahajournals.org">http://ahajournals.org</a> by on March 13, 2025 Lawton et al 2021<br>
ACC/AHA/SCAI Coronary Revascularization Guideline Circulation.<br>
2022;145:e18–e114. DOI: 10.1161/CIR.0000000000001038 January 18, 2022<br>
e89 CLINICAL STATEMENTS AND GUIDELINES tected left main stenosis:<br>
updated 5-year outcomes from the randomised, non-inferiority NOBLE<br>
trial. Lancet. 2020;395:191–199. 52. Kuno T, Ueyama H, Rao SV, et al.<br>
Percutaneous coronary intervention or coronary artery bypass graft<br>
surgery for left main coronary artery disease: a meta-analysis of<br>
randomized trials. Am Heart J. 2020;227:9–10. 53. Park D-W, Ahn J-M,<br>
Park H, et al. Ten-year outcomes after drug-eluting stents versus<br>
coronary artery bypass grafting for left main coronary disease:<br>
extended follow-up of the PRECOMBAT trial. Circulation. 2020;141:1437–<br>
1446. 54. Ahmad Y, Howard J P, Arnold AD, et al. Mortality after drug-eluting stents vs. coronary artery bypass grafting for left main<br>
coronary artery disease: a meta-analysis of randomized controlled<br>
trials. Eur Heart J. 2020;41:3228– 3235.</p>
</blockquote>
<p>Nội dung trên chỉ là phần trích dẫn các tài liệu tham khảo, nhưng tên các tài liệu thường chứa nhiều keywords mà ta cần tìm vậy nên chúng có thể bị trả về, dù thông tin hoàn toàn vô dụng. RAG về bản chất không có trí thông minh, nó không biết được cái nào là quan trọng, vậy nên ta cần chiến thuật prompting phù hợp (prompt cho cái search chứ không phải LLM).</p>
<p>Thử nghiệm prompt đầu tiên:</p>
<blockquote>
<p>Record: Patient: John Smith Age: 58, Male Medical History: Diabetes;<br>
multivessel coronary artery disease with left anterior descending<br>
(LAD) involvement Vital Signs: BP 140/85 mmHg, HR 80 bpm Laboratory<br>
Findings: Hemoglobin 9.0 g/dL Preoperative Workup: Basic clinical<br>
assessment, coronary angiography Task: Identify laboratory results<br>
outside of normal reference ranges</p>
</blockquote>
<p>Thì cho kết quả như trên.<br>
Sau khi tìm hiểu một hồi thì vấn đề có vẻ là tại RAG cũng không hiểu cả ngôn ngữ tự nhiên (do nó không dùng LLM). Thế nên em đổi sang dùng <strong>từ khóa</strong>, sử dụng <strong>Llama-3.2-3B</strong> để trích xuất các từ đó ra cùng một dòng rồi mới ném vào RAG. Cách này đạt hiệu quả cao hơn, theo một số thử nghiệm của em, nhưng nó cũng hên xui, tại các phần mục lục hoặc tham khảo cũng chứa rất nhiều keywords. Em có thử sử dụng filter (ví dụ, task 1 chỉ được nhận kết quả từ tài liệu X) thì cách này hoạt động khá tốt. Nhưng mục tiêu của ta là cần đọc cả 4 tài liệu và trả lời, dù sao thì tiểu sử bệnh nhân có thể thay đổi và có thể cần thông tin từ nhiều tài liệu khác nhau.</p>
<p>Giải pháp của em là tăng số k_neighbors từ 1 hoặc 5 lên 10. k càng lớn thì càng có xác suất chọn trúng đối tượng cần tìm (giống paper zero-shot prompting?). Và lần này thay vì trích xuất từ khóa, thì em ra lệnh cho con chatbot tóm tắt lại yêu cầu và nêu từ khóa, nội dung quan trọng một cách ngắn gọn. Kết quả khá vượt ngoài mong đợi, RAG có trả về được văn bản liên quan, nhưng em chỉ đánh giá được với task 1, còn các task sau thì nặng về chuyên môn quá nên em cũng không biết. Nếu có database để đánh giá chính xác hơn thì sẽ tốt, nhưng hiện tại model của em hoạt động có vẻ sát với mong đợi.</p>

