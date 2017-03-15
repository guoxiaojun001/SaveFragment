# SaveFragment
关于fragment状态保存和恢复问题
<p>
http://blog.csdn.net/jianghejie123/article/details/44726691
原文链接:http://www.jianshu.com/p/75dc2f51cd63
Fragment 最终模版

如下就是我现在用于我工作中的 fragment 模版。

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.inthecheesefactory.thecheeselibrary.R;

/**
 *Created by nuuneoi on 11/16/2014.
 */
public class StatedFragment extends Fragment {

    Bundle savedState;

    public StatedFragment() {
        super();
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        // Restore State Here
        if (!restoreStateFromArguments()) {
            // First Time, Initialize something here
            onFirstTimeLaunched();
        }
    }

    protected void onFirstTimeLaunched() {

    }

    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        // Save State Here
        saveStateToArguments();
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        // Save State Here
        saveStateToArguments();
    }

    ////////////////////
    // Don't Touch !!
    ////////////////////

    private void saveStateToArguments() {
        if (getView() != null)
            savedState = saveState();
        if (savedState != null) {
            Bundle b = getArguments();
            b.putBundle("internalSavedViewState8954201239547", savedState);
        }
    }

    ////////////////////
    // Don't Touch !!
    ////////////////////

    private boolean restoreStateFromArguments() {
        Bundle b = getArguments();
        savedState = b.getBundle("internalSavedViewState8954201239547");
        if (savedState != null) {
            restoreState();
            return true;
        }
        return false;
    }

    /////////////////////////////////
    // Restore Instance State Here
    /////////////////////////////////

    private void restoreState() {
        if (savedState != null) {
            // For Example
            //tv1.setText(savedState.getString("text"));
            onRestoreState(savedState);
        }
    }

    protected void onRestoreState(Bundle savedInstanceState) {

    }

    //////////////////////////////
    // Save Instance State Here
    //////////////////////////////

    private Bundle saveState() {
        Bundle state = new Bundle();
        // For Example
        //state.putString("text", tv1.getText().toString());
        onSaveState(state);
        return state;
    }

    protected void onSaveState(Bundle outState) {

    }
}
如果你使用这个模版，仅仅只需简单地继承这个类StatedFragment，然后在 onSaveState() 中保存事物，在 onRestoreState() 中恢复它们。上述代码就会为你做好剩下的工作，我相信这已经覆盖了我已知的可能情况。

setRetainInstance 能够帮助开发者在布局改变时（如：屏幕旋转）处理成员变量 而你可能注意到我没有设置 setRetaionInstance 为 true。请记住，这就是我的目的，因为setRetainInstance(true)并没有覆盖全部的情况。最主要的原因是你不能一次又一次地保存那些经常背使用的嵌套 fragment 。所以我建议就是不要保存实例，除非你100%确定这个 fragment 不会用于嵌套。

用法

好消息。这个博客讲述的 StateFragment 现在加入了一个非常容易使用的库，现在已经在 jcenter 上发布了。现在你可以简单地在你工程的 build.gradle 文件中加上一个依赖。如下所示：

dependencies {
    compile 'com.inthecheesefactory.thecheeselibrary:stated-fragment-support-v4:0.9.1'
}
继承 StateFragment ，然后在 onSaveState(Bundle outState) 中保存状态，在 onRestoreState(Bundle saveInstanceState)中恢复状态。如果你想在 fragment 第一次启动时做点什么的话，你也可以覆盖 onFirstTimeLaunched() 方法（在之后不会被调用）。

public class MainFragment extends StatedFragment {

    ...

    /**
     * Save Fragment's State here
     */
    @Override
    protected void onSaveState(Bundle outState) {
        super.onSaveState(outState);
        // For example:
        //outState.putString("text", tvSample.getText().toString());
    }

    /**
     * Restore Fragment's State here
     */
    @Override
    protected void onRestoreState(Bundle savedInstanceState) {
        super.onRestoreState(savedInstanceState);
        // For example:
        //tvSample.setText(savedInstanceState.getString("text"));
    }

    ...

}
