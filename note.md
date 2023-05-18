## Project setup
- xoa App.css, App.test.js, index.css, logo.svg, reportWebVitals.js, setupTests.js
- xoa dong import './index.css', import reportWebVitals from './reportWebVitals'; va cac dong tu comment tro di trong index.js
- xoa dong import logo from './logo.svg';
import './App.css'; tat ca moi thu trong cau lenh return trong App.js
- install quill vao thu muc client
- tao them 1 component ten TextEditor.js
- dung lenh rfc de tao nhanh react component trong TextEditor.js
- import TextEditor vao App.js, return no ra
## Setting Up Quill
- do trong bai nay su dung snow theme nen import no vao component TextEditor + import Quill vao TextEditor
- Quill duoc import vao chua phai la 1 react component, phai tao instance bang new Quill
- do Quill nay se duoc render vao duy nhat 1 lan nen su dung useEffect voi tham so thu 2 la [] (useEffect luon duoc goi moi khi component duoc render xong, co the tra ve 1 ham ham nay se duoc goi khi component do unmount) - su dung useEffect thi phai import no vao
- khi component bi render ra 2 lan thi loi do React.strictmode
- khi trong code co thay doi, moi lan save se render them 1 toolbars, dieu nay la do chua clean up toolbar, quill tao ra text editor ben trong element ta chon nhung tao toolbar ben ngoai
- de khac phuc chung ta co the tao element div, tao 1 tham chieu toi element ta chon bang useRef() sau do truyen bien do vao element bang ref=... cai ref do .current.append element div ma chung ta vua moi tao ra, sau do dung element div do de lam vat chua cho editor, return lai bang innerHTML rong
- loi tiep theo trong 1 vai tinh huong, khi reload lai trang thi tham chieu ref do khong ton tai nua nen khong the su dung innerHTML, k hieu lam co ma no la v :))
- cach giai quyet la su dung useCallback (useCallback dung de ghi nho 1 ham khi cac dependencies khong thay doi)
- tao ra 1 ham tra ve tu useCallback, nhan vao 1 doi so la element container, thuc thi nhung logic trong do, k can return vi day k phai useEffect, kiem tra trong dieu kien container null, truyen ham vao ref cua element container, khi do moi khi container duoc mount se goi toi ham ben trong ref la ham dc tao tu useCallback
## CSS Styles
- tao file style.css trong thu muc src, import file nay vao index.js
- them cac tuy chon cho toolbar = 1 array, moi phan tu trong do la 1 array
- [{ header [1, 2, 3, 4, 5, 6, false] }]: tao 6 loai heading
- [{ font: [] }]: font rong, su dung font mac dinh
- them vao option tool bar bang modules: { toolbar: ...}
## Setup server
- cai dat socket.io
- tao bien io, doi so la cong + option, trong option tao 1 option cors voi origin la dia chi chay cua client, methods la post + get
- io.on connection de xu li viec ket noi cua client
- cai dat socket.io-client cho phia client
- import io vao client o TextEditor file, su dung useEffect de ket noi toi socket o server
- io voi doi so la dia chi cua io server
- return ham disconnect de don
- phia server cai dat them nodemon, tao cau len start bang nodemon
- de dong bo nhung gi duoc viet tren docs, su dung useState doi voi socket + quill de co bien xai
- su dung useEffect moi khi socket hoac quill thay doi, xu li bang api cua quill on text-change truyen vao 1 ham nhan 3 doi so delta, olddenta, source, kiem tra neu la nguoi dung thay doi thi moi emit su kien gui du lieu do ve phia server (kiem tra xem co quill vs socket k de tranh loi)
- xu li su kien on send-changes phia server trong ham callback cua io.on, phat ra 1 su kien de cap nhat tren cac client khac su dung socket.broadcast de gui toi tat ca nhung client khac tru cua minh
- phia client dung useEffect xu li su kien cap nhat, quill.updateContents truyen delta vao de cap nhat
## Handling Multiple Documents
- cai dat react-router-dom vao client, import browserrouter, switch, route, redirect vao App.js
- Dua TextEditor vao Router > Switch > Route path=/documents/:id, tren la Route path / exact, trong path / redirect to /documents/${uuidV4()}
- Cai dat uuidV4 bang npm i uuid, import vao bang import v4
- Trong TextEditor.js, import useParam vao, lay documentId bang useParams(), su dung useEffect khi socket, quill, documentId thay doi
- Neu null thi nhu thuong le, socket.once (tu dong xoa cai lang nghe nay khi da lang nghe xong) de kich hoat xu li su kien 1 lan duy nhat 'load-document', setContents = document sau do enable de co the viet vao
- emit ra su kien get-document truyen vao documentId de lay document tu phia server
- O ham chinh su dung disable Quill vs setText loading de xu li khi dang tim document
- O phia server, lang nghe su kien get-document, tam cho data = "", socket.join de dua client do vao phong documentId , tai cho broadcast them .to(documentId) de gui + nhan tin nhan dua tren phong do
- Phat ra su kien load-document, truyen data ra ngoai cho phia client load, dua socket broadcast cac thu vao trong cai lang nghe su kien get-document
## persisting documents
- cai dat mongoose, require + setting theo docs vao server.js
- Tao scheme Document.js
- import Schema model tu mongoose
- Tao schema co _id la String, data la Object
- export model("Document", Document)
- import schema vao server.js
- Viet ham fineOrCreateDocument(id) neu id null thi return, await Document.findById, neu Document co thi return con k thi await Document.create({})
- thay the cac vi tri trong lang nghe su kien cho dung
- viet them su kien save-document ham asynce await findByIdAndUpdate
- vao TextEditor, viet them useEffect, setInterval sau 1 khoang thoi gian tu dong phat ra su kien save-document