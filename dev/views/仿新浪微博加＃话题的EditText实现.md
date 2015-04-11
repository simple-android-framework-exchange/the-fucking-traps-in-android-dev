# 仿新浪微博加＃话题的EditText实现
----
作者 : [Ailurus](https://github.com/liangzhitao)

##需求
产品上线了，项目差不多算是稳定下来了，接下来就是一个个的版本迭代了。这周又增加了几个新功能，其中一个就是题目中讲的，要仿新浪微博（如下图）的输入框里的文字效果。作为整体的字体两边是井号，并且包括井号要变色，删除的时候，当光标到右侧的井号，再次点击删除的时候，会将井号包裹的内容作为一个整体删除掉，同时焦点不能在变色的字符之间，也就是说当点击变色的字符时，光标会自动地落在井号两侧。
![](http://easydone.qiniudn.com/weibo_EditText_1.png)

##拆分问题
看上去是一个挺简单合理的需求，随便一想，这不就是字符串匹配嘛！可是实际行动起来，就会发现各种坑爹问题，远不是想象中的那么简单。我这做下来的感受是，必须要有一个明确清晰的思路，总结下来，其实可以分为下面几个步骤。

> * 首先，新浪微博这个功能的入口是，点击井号，进入到新的界面里选择字符串，然后自动**添加**到 EditText 框里，我们要实现这样一个 Demo ，也没必要进入新的界面，直接做一个按钮，点击添加即可；

> * 然后，需要明确的一点是，在 Android 提供的原生的 API 里，**删除**动作，一次只能删掉一个字符，而无法删除批量的字符，一次只能删除一个 letter ，而不是一个 word ，删除动作是我们这个课题的关键所在；

> * 处理完了增加和删除字符串功能，接着就是匹配符合规则的字符串做**变色**处理了；

> * 再接着就是处理点击字符串，光标的所处位置的问题；

> * **完善**需要补充和修改的细节。

通过以上五个步骤将课题拆分为四个小问题，然后再去一一解决即可。

##添加字符串
这一步基本上算是最简单的一步了。我们只需要做到点击一个 button ，将字符串 *append* 到 EditText 里就可以了。当然，为了让我们的操作更加有广泛的适用性，单纯地增加字符串就有点太不够看了，而实际应用里，这样做基本上也没有什么意义。

我们要做的就是，创建一个实体类，而这个字符串作为一个值，所对应的变量就是这个实体类的成员变量，我们通过不断往实体类集合里添加 new 出来的对象，来往 EditText 里追加字符串。同时，这样做还有一个好处就是，实际应用里，往往伴随字符串使用的可能还有其他的变量，要处理与字符串一一对应的其他变量，最好的方法就是在增加和删除字符串的同时，将字符串对应变量所在实体从实体集合中移除掉。

下面就是我的实体类：
```Java
/** 实体类 */
public class BookEntity implements Serializable {
	private static final long serialVersionUID = 1L;
	private String bookId;
	private String bookName;
    public BookEntity(String bookName, int bookId) {
		super();
		this.bookName = bookName;
		this.bookId = bookId;
	}
	public String getBookId() {
		return bookId;
	}
	public void setBookId(String bookId) {
		this.bookId = bookId;
	}
	public String getBookName() {
		return bookName;
	}
	public void setBookName(String bookName) {
		this.bookName = bookName;
	}
}
```
添加字符操作：
```Java
/** 执行增加字符串的操作 */
private Button button;
private ArrayList<BookEntity> mList = new ArrayList<BookEntity>();
private EditText editText;
View view = inflater.inflate(R.layout.fragment_main, container, false);
editText = (EditText) view.findViewById(R.id.edit_text);
button = (Button) view.findViewById(R.id.bt);
buttont.setOnClickListener(new OnClickListener() {
	@Override
	public void onClick(View v) {
		int nextInt = new Random().nextInt(100);
		String str = "#测试测试" + nextInt + "# ";
		editText.setText(editText.getText());
		editText.append(str);
		editText.setSelection(editText.getText().toString().length());
		mList.add(new BookEntity(str, nextInt));
	}
});
```

这样我们增加字符串操作就做完了。

##删除符合条件的字符串
删除操作是这个小程序的关键。做删除操作时，我们需要搞清楚这样几个问题：什么时候删除？从哪儿开始删？删到哪里算结束？删除结束之后，光标位置应该在哪里？接下来一个一个解决。

1. 删除操作当然应该在按下软键盘的删除键时执行，这里有一个细节，在 Android 点击键盘按键的事件，其实是分开处理的，按下键盘时会出发动作，弹起键盘时同样会触发动作；
2. 如果是不符合筛选条件（"#"+bookEntity.getBookName()+"#"），点击一次就删除一个字符；如果符合条件，就一次性将整个带左右井号的字符串都删掉；
3. 而光标则应该一直保持在最末尾处。

```Java
/** 监听删除按键，执行删除动作 */
editText.setOnKeyListener(new OnKeyListener() {
	@Override
	public boolean onKey(View v, int keyCode, KeyEvent event) {
		if (keyCode == KeyEvent.KEYCODE_DEL && event.getAction() == KeyEvent.ACTION_DOWN) { //当为删除键并且是按下动作时执行
			int selectionStart = editText.getSelectionStart();
			int lastPos = 0;
			for (int i = 0; i < mList.size(); i++) { //循环遍历整个输入框的所有字符
				if ((lastPos = editText.getText().toString().indexOf(mList.get(i).getBookName(), lastPos)) != -1) {
					if (selectionStart >= lastPos && selectionStart <= (lastPos + mList.get(i).getBookName().length())) {
						String sss = editText.getText().toString();
						editText.setText(sss.substring(0, lastPos) + sss.substring(lastPos + mList.get(i).getBookName().length())); //字符串替换，删掉符合条件的字符串
						mList.remove(i); //删除对应实体
						editText.setSelection(lastPos); //设置光标位置
						return true;
					}
				} else {
						lastPos += ("#" + mList.get(i).getBookName() + "#").length();
				}
			}
		}
		return false;
	}
});
```

##处理变色问题
变色，使符合条件的字符串颜色高亮，更多像是锦上添花的功能，给用户更直观的感知，确实整个小程序里实现起来最复杂的部分。由于 Android 自身的一些原因，无论是增加还是删除，或者是锁屏等事件，都会造成界面重绘的问题，因此其核心就在于要设置**字符改变**的监听状态，当字符改变时，剩余的字符的颜色随之发生变化。

具体的实现，我们需要一个 TextWatcher 的实现类，然后 new 出来一个对象，作为参数给控件 editText 设置的 addTextChangedListener 监听。
```Java
class MyTextWatcher implements TextWatcher {
	@Override
	public synchronized void afterTextChanged(Editable s) {
		AddNewArticleUI.this.etWriteDynamic.removeTextChangedListener(watcher);
		TEXT_CHANGE_LISTENER_FLAG = 0;
		int findPos = 0;
		int copyPos = 0;
		String sText = s.toString();
		List<Integer> spanIndexes = new ArrayList<Integer>();
		s.clear();
		for (int i = 0; i < bookList.size(); i++) {
			String tempBookName = "#" + bookList.get(i).getBookName() + "#";
			if ((findPos = sText.indexOf(tempBookName, findPos)) != -1) {
				spanIndexes.add(findPos);//bookName 的开始索引，键值为偶数，从0开始
				spanIndexes.add(findPos + tempBookName.length()); //bookName 的结束索引，键值为奇数，从1开始
			}
		}
		if (spanIndexes != null && spanIndexes.size() != 0) {
			for (int i = 0; i < spanIndexes.size(); i++) {
				if (i % 2 == 0) {
					s.append(sText.substring(copyPos, spanIndexes.get(i)));
				} else {
					Spanned htmlText = Html.fromHtml("<font color='blue'>" + sText.substring(copyPos, spanIndexes.get(i)) + "</font>");
					s.append(htmlText);
				}
				copyPos = spanIndexes.get(i);
			}
			s.append(sText.substring(copyPos));
		} else {
			s.append(sText);
		}
	}

	@Override
	public void beforeTextChanged(CharSequence s, int start, int count, int after) {}

	@Override
	public void onTextChanged(CharSequence s, int start, int before, int count) {}
}
```
这里需要注意的是在 addTextChangedListener 的 afterTextChange 方法里，不能够操作 Editable 去给控件设置值，而且也不能在全局设置 TextChanged 监听，否则必然会因为循环调用 StackOverFlow 。也就是说我们只能在文字改变之前去调用，还要发屏幕解锁屏广播，在屏幕解锁时去调用。

```Java
private int TEXT_CHANGE_LISTENER_FLAG = 0;
/** 监听文字变化，并重新设置颜色 */
if (TEXT_CHANGE_LISTENER_FLAG == 0) {
	editText.addTextChangedListener(watcher);
	TEXT_CHANGE_LISTENER_FLAG = 1;
}
```
下面是解锁屏的广播（此段代码来自网络）
```Java
public class ScreenListener {
	private Context mContext;
	private ScreenBroadcastReceiver mScreenReceiver;
	private ScreenStateListener mScreenStateListener;

	public ScreenListener(Context context) {
		mContext = context;
		mScreenReceiver = new ScreenBroadcastReceiver();
	}

	/**
	 * screen状态广播接收者
	 */
	private class ScreenBroadcastReceiver extends BroadcastReceiver {
		private String action = null;

		@Override
		public void onReceive(Context context, Intent intent) {
			action = intent.getAction();
			if (Intent.ACTION_SCREEN_ON.equals(action)) { // 开屏
				mScreenStateListener.onScreenOn();
			} else if (Intent.ACTION_SCREEN_OFF.equals(action)) { // 锁屏
				mScreenStateListener.onScreenOff();
			} else if (Intent.ACTION_USER_PRESENT.equals(action)) { // 解锁
				mScreenStateListener.onUserPresent();
			}
		}
	}

	/**
	 * 开始监听screen状态
	 * 
	 * @param listener
	 */
	public void begin(ScreenStateListener listener) {
		mScreenStateListener = listener;
		registerListener();
		getScreenState();
	}

	/**
	 * 获取screen状态
	 */
	private void getScreenState() {
		PowerManager manager = (PowerManager) mContext.getSystemService(Context.POWER_SERVICE);
		if (manager.isScreenOn()) {
			if (mScreenStateListener != null) {
				mScreenStateListener.onScreenOn();
			}
		} else {
			if (mScreenStateListener != null) {
				mScreenStateListener.onScreenOff();
			}
		}
	}

	/**
	 * 停止screen状态监听
	 */
	public void unregisterListener() {
		mContext.unregisterReceiver(mScreenReceiver);
	}

	/**
	 * 启动screen状态广播接收器
	 */
	private void registerListener() {
		IntentFilter filter = new IntentFilter();
		filter.addAction(Intent.ACTION_SCREEN_ON);
		filter.addAction(Intent.ACTION_SCREEN_OFF);
		filter.addAction(Intent.ACTION_USER_PRESENT);
		mContext.registerReceiver(mScreenReceiver, filter);
	}

	public interface ScreenStateListener {// 返回给调用者屏幕状态信息
		public void onScreenOn();

		public void onScreenOff();

		public void onUserPresent();
	}
}
```
注册广播
```Java
ScreenListener screenListener = new ScreenListener(this);
screenListener.begin(new ScreenStateListener() {

    @Override
    public void onUserPresent() {
        Log.e("onUserPresent", "onUserPresent");
    }

    @Override
    public void onScreenOn() {
        Log.e("onScreenOn", "onScreenOn");
        if (TEXT_CHANGE_LISTENER_FLAG == 0) {
			editText.addTextChangedListener(watcher);
			TEXT_CHANGE_LISTENER_FLAG = 1;
		}
    }

    @Override
    public void onScreenOff() {
        Log.e("onScreenOff", "onScreenOff");
    }
});
```

最后，不要忘记取消注册广播。

##设置光标位置

最后就是在手指触摸到所匹配的字符时，设置光标的位置，其实跟上面的删除处理逻辑一样，只是不需要再去替换字符串，判断的逻辑都是一样的。

```Java
editText.setOnClickListener(new OnClickListener() {
	@Override
	public void onClick(View v) {
		Log.i("TAG", ((EditText) v).getSelectionStart() + "");
		int selectionStart = ((EditText) v).getSelectionStart();
		int lastPos = 0;
		for (int i = 0; i < mList.size(); i++) {
			if ((lastPos = editText.getText().toString().indexOf(mList.get(i).getBookName(), lastPos)) != -1) {
				if (selectionStart >= lastPos && selectionStart <= (lastPos + mList.get(i).getBookName().length())) {
					editText.setSelection(lastPos + mList.get(i).getBookName().length());
				}
			} else {
					lastPos += ("#" + mList.get(i).getBookName() + "#").length();
			}
		}
	}
});
```

至此，一个类似新浪微博输入框的小程序就完成了。当然比起新浪微博，这里少了一步点击删除第一次选中被匹配到字符串，这个功能，新浪微博的实现是，点击删除，先选中字符串，再点击删除，就删掉字符串。具体情况，不再赘述。