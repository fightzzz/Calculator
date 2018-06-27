import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.GridLayout;
import java.awt.SystemColor;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.text.NumberFormat;
import java.util.regex.Pattern;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JMenu;
import javax.swing.JMenuBar;
import javax.swing.JPanel;
import javax.swing.border.BevelBorder;

public class Calculator {
    // 主框架
    private JFrame jframe = new JFrame("Calculator by Group 22");
    //显示窗口载体
    private JPanel jpanel_show = new JPanel();
    // 输入显示标签
    private JLabel jla_input = new JLabel();
    // 结果显示标签
    private JLabel jla_show = new JLabel();
    // 错误信息显示标签
    private JLabel jla_error = new JLabel();
    // 计算器按键载体
    private JPanel jp_button = new JPanel();
    private String result = " ";
    // 按键显示字符
    private String[] str = { "1", "2", "3", "+", "x²","x³", "4", "5", "6", "-", "Abs","cos", "7", "8", "9", "*", "1/x","tan", "0", "(",
            ")", "÷", "log","sin",".","←", "c", "=", "Int","√"};
    // 按键
    private JButton[] jb_button = new JButton[str.length];

    /**
     * 界面初始化
     */
    private void init() {

        // 设置显示框计算结果大字体, 设置字体右对齐
        jla_show.setText("0");
        jla_show.setHorizontalAlignment(JLabel.RIGHT);
        jla_show.setFont(new Font("arial", Font.BOLD, 22));
        jla_show.setForeground(Color.BLUE);
       

        // 设置显示框输入显示的字体,设置字体右对齐
        jla_input.setText(" ");
        jla_input.setHorizontalAlignment(JLabel.RIGHT);
        jla_input.setFont(new Font("", Font.ROMAN_BASELINE, 10));

        // 设置错误信息显示
        jla_error.setText(" ");
        jla_error.setHorizontalAlignment(JLabel.RIGHT);
        jla_error.setFont(new Font("arial", Font.ROMAN_BASELINE, 10));
        jla_error.setForeground(Color.RED);

        jpanel_show.setLayout(new BorderLayout());

        // 显示标签加载到显示窗口载体中
        jpanel_show.add(jla_show, BorderLayout.SOUTH);
        jpanel_show.add(jla_input, BorderLayout.CENTER);
        jpanel_show.add(jla_error, BorderLayout.NORTH);

        jpanel_show.setBorder(
                new BevelBorder(BevelBorder.RAISED, new Color(250, 250, 250), null, SystemColor.scrollbar, null));

        // 按键设置为网格布局
        jp_button.setLayout(new GridLayout(5, 6, 4, 4));
        jp_button.setBorder(
                new BevelBorder(BevelBorder.RAISED, new Color(180, 180, 180), null, SystemColor.scrollbar, null));
        // 按键
        for (int i = 0; i < jb_button.length; i++) {
            jb_button[i] = new JButton(str[i]);
            jb_button[i].setContentAreaFilled(false);// 除去默认的背景填充
            jb_button[i].setFocusPainted(false);// 除去焦点的框
            
            // 按键注册监听器
            jb_button[i].addActionListener(new ButtonListener());
            // 按键添加到按键载体中
            jp_button.add(jb_button[i]);
        }
        // 按键载体添加到主框架
        jframe.add(jp_button, BorderLayout.CENTER);
        // 显示框载体加载到主框架
        jframe.add(jpanel_show, BorderLayout.NORTH);
       

        // 窗口位置，左上角为屏幕中央
        jframe.setLocationRelativeTo(null);
        // 自适应大小
        jframe.pack();
        // 设置X关闭
        jframe.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // 不可放大
        jframe.setResizable(false);
        // 显示窗口
        jframe.setVisible(true);

    }

    /**
     * 事件处理
     * 
     *
     *
     */
    private class ButtonListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            String content = String.valueOf(e.getActionCommand());
            boolean flag1 = jla_input.getText().contains("=");
            // 输入处理的结果赋值给result
            result = parseInputStr(result, content);
            jla_input.setText(result);
            // 控制计算结果后继续输入
            if (flag1 && content.matches("^\\d")) {
                result = content;
                jla_input.setText(result);
                
            }
            if (content.equals("←"))
                backSpace();// 后退

            if (content.equals("c"))
                clear();// 清屏

            if (result.endsWith("=")) {
                String str = jla_input.getText();
                // 如果（大于）数，报错“缺少）”
                if (countsOfSubstringInString(result, "(") > countsOfSubstringInString(result, ")")) {
                    jla_error.setText("')' is missing!");
                    result = result.substring(0, result.length() - 1);

                }
                else if (str.indexOf("log") != -1) {
                    int index = indexOfOpecifiedMirrorBracket(str, '(',
                            str.indexOf("log") + 3);
                    String s = str.substring(str.indexOf("log") + 4, index);
                    if (parse(s) < 0.0) {
                        jla_error.setText("negative number do not available!");
                    } else
                        calculate();
                }
                // 如果出现根号下负数，报错“负数没有平方根”
                else if (str.indexOf("sqt") != -1) {
                    int index = indexOfOpecifiedMirrorBracket(str, '(',
                            str.indexOf("sqt") + 3);
                    String s = str.substring(str.indexOf("sqt") + 4, index);
                    if (parse(s) < 0.0) {
                        jla_error.setText("negative number do not have a square root!");
                    } else
                        calculate();
                }
                // 如果出现/0.0报错“0不能作为被除数”
                else if (jla_input.getText().indexOf("/") != -1) {
                    String s = "";                  
                    int indexofdiv = indexOfNumberEnd(str, str.indexOf("/"));
                    s = str.substring(str.indexOf("/") + 1, indexofdiv);
                    if (parse(s) == Double.parseDouble("0")) {// 如果s=0
                        jla_error.setText("0 can not be the dividend!");
                        result = result.substring(0, result.length() - 1);
                    } else
                        calculate();
                } else
                    calculate();
            }
        }
    }

    private void calculate() {
        result = String.valueOf(parse(result.substring(0, result.length() - 1)));
        jla_show.setText(formatResult(result));
    }

    // 处理显示结果
    private String formatResult(String result) {
        String finalResult = "";
        Double d = Double.valueOf(result);
        NumberFormat nf = NumberFormat.getInstance();
        nf.setGroupingUsed(false);
        if (Math.abs(d) >= 1 || Math.abs(d) == 0)
            finalResult = nf.format(d);
        else
            finalResult = d.toString();
        return finalResult;
    }

    /**
     * 后退处理
     * 
     * @param content
     */
    private void backSpace() {
        if (result.length() == 1 || result.equals("")) {
            result = " ";
            jla_input.setText(result);
        } else {
            result = result.substring(0, result.length() - 1);
            if (result.endsWith(" "))
                result = result.substring(0, result.length() - 1);
            jla_input.setText(result);
        }
        jla_error.setText(" ");
    }

    /**
     * 清屏处理
     * 
     * @param content
     */
    private void clear() {
        jla_error.setText(" ");
        jla_input.setText(" ");
        result = " ";
        jla_show.setText("0");
    }

    public static void main(String[] args) {
        new Calculator().init();
    }

    /**
     * 输入处理
     * 
     * @param result
     * @param content
     * @return
     */
    private static String parseInputStr(String result, String content) {

        // 刚开始输入只能输入数字-或者(
        if (result.equals(" ") || result.equals("")) {
            if (content.matches("\\d+|\\-|\\(*")) {
                result = "".concat(content);
            }
        }

        // 左括号数大于右括号数才能输入右括号
        else if (content.equals(")")) {
            boolean b = result.matches(".*\\d$|.*\\)$");// result以数字结尾或者右括号结尾
            if (countsOfSubstringInString(result, "(") > countsOfSubstringInString(result, ")") && b)
                result = result.concat(content);
            else
                result = result.concat("");
        }

        // 以数字结尾
        else if (Pattern.matches(".*\\d$", result)) {
            // 0后不可输入数字
            if (lastNumberOfStr(result).matches("^0+")
                    &&  !content.matches("\\+|\\-|\\*|÷|\\.|=|log|sin|cos|tan|x²|x³|1/x|√"))
                result = result + "";
            // 数字后面跟x²，1/x，√
            else if (content.matches("x²|x³|1/x|√|log|sin|cos|Abs|Int")) {
                result = parseDigitEndWithFuncthion(result, content);
            }
            // x.y后不能接小数点，如2.1后不能输入点，不能出现2.1.2
            else if (Pattern.matches(".*\\.\\d+$", result)) {// resul以.数字结尾
                result = result.concat(content.matches("\\d*|\\+|\\-|\\*|÷|\\)|=") ? content : "");
            }
            // 数字或小数点
            else if (content.matches("\\d|\\."))
                result = result.concat(content);
            else
                result = result.concat(content.matches("\\+|\\-|\\*|÷|\\)|=") ? " " + content : "");
        }
        // 以小数点结尾
        else if (Pattern.matches(".*\\.$", result) && Pattern.matches("^\\d", content)) {
            result = result.concat(content);
        }
        // 以左括号结尾
        else if ((result).matches(".*\\($")) {
            result = result.concat(content.matches("\\d*|\\-|\\(") ? content : "");
        }
        // 以右括号结尾
        else if (Pattern.matches(".*\\)$", result)) {
            if (content.matches("\\+|\\-|\\*|÷|=")) {
                result = result + " " + content;
            }
            if (content.matches("\\)")) {
                result = result + content;
            }
            if (content.matches("x²|x³|1/x|√|log|sin|cos|Abs|Int")) {
                result = parseBrackets(result, content);
            }
        }
        // 以加减乘除结尾
        else if (result.matches(".*\\+$|.*\\-$|.*\\*$|.*÷$|.*/$")) {
            result = result.replaceAll("÷", "/");
            if (result.matches(".*\\-$")) {// 以减号结尾
                if (result.length() == 1)
                    result = result + (content.matches("\\(|\\d") ? content : "");// 负号
                else if (result.length() > 1) {// 减号或左括号负号
                    boolean b = result.charAt(result.length() - 2) == '(';
                    if (b)
                        result = result + (content.matches("\\(|\\d") ? content : "");// 左括号负号
                    else
                        result = result + (content.matches("\\(|\\d") ? " " + content : "");// 减号
                }
            } else
                result = result + (content.matches("\\(|\\d") ? " " + content : "");
        }
        
      
        return result;
    }

    /**
     * 计算处理
     * 
     * @param content
     * @return
     */
    private static double parse(String content) {

       
      
        if (content.contains("1/x"))
            content = content.replace("1/x", "rec");
       
        // 处理平方
        int index = content.indexOf("sqr");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            String d = "(" + content.substring(index + 3, endindex + 1) + "*"
                    + content.substring(index + 3, endindex + 1) + ")";
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }
        // 处理立方
        index = content.indexOf("cube");
       if (index != -1) {
           int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 4);
          
           String d = "(" + content.substring(index + 4, endindex + 1) + "*"
                   + content.substring(index + 4, endindex + 1) + "*"+content.substring(index + 4, endindex + 1)+")";
           return parse(content.substring(0, index) + d + content.substring(endindex + 1));
       }

        // 处理开方
        index = content.indexOf("sqt");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = Math.sqrt(parse(content.substring(index + 3, endindex + 1)));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }
        index = content.indexOf("Abs");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = Math.abs(parse(content.substring(index + 3, endindex + 1)));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }
        //处理取整
        index = content.indexOf("Int");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = Math.floor(parse(content.substring(index + 3, endindex + 1)));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }
        // 处理sin
        index = content.indexOf("sin");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = Math.sin(parse(content.substring(index + 3, endindex + 1)));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }

        // 处理cos
        index = content.indexOf("cos");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = Math.cos(parse(content.substring(index + 3, endindex + 1)));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }

        // 处理求倒
        index = content.indexOf("rec");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = parse("1/" + content.substring(index + 3, endindex + 1));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }

        index = content.indexOf("log");
        if (index != -1) {
            int endindex = indexOfOpecifiedMirrorBracket(content, '(', index + 3);
            double d = Math.log10(parse(content.substring(index + 3, endindex + 1)));
            return parse(content.substring(0, index) + d + content.substring(endindex + 1));
        }
        // 处理括号
        int startindex = content.indexOf("(");
        boolean b = countsOfSubstringInString(content, "(") == countsOfSubstringInString(content, ")");
        if (startindex != -1 && b) {
            int endindex = indexOfFirstMirrorBracket(content, '(');
            double d = parse(content.substring(startindex + 1, endindex));
            return parse(content.substring(0, startindex) + d + content.substring(endindex + 1));
        }

        


        // 处理加法
        index = content.indexOf("+");
        if (index != -1) {
            return parse(content.substring(0, index)) + parse(content.substring(index + 1));
        }

        // 处理减法
        index = content.lastIndexOf("-");
        if (index != -1) {
            return parse(content.substring(0, index)) - parse(content.substring(index + 1));
        }

        // 处理乘法
        index = content.indexOf("*");
        if (index != -1) {
            return parse(content.substring(0, index)) * parse(content.substring(index + 1));
        }

        // 处理除法
        index = content.lastIndexOf("/");
        if (index != -1) {
            return parse(content.substring(0, index)) / parse(content.substring(index + 1));
        }

        // 处理空格
        if (content.equals("") || content.equals(" "))
            content = "0";

        // 最后结果
        return Double.parseDouble(content);
    }

    /**
     * 字符串中某子字符串的数量
     * 
     * @param string
     * @param substring
     * @return
     */
    private static int countsOfSubstringInString(String string, String substring) {
        int sum = 0;
        String subStrHead = "";
        String subStrTrail = "";
        while (string.contains(substring)) {
            sum++;
            subStrHead = string.substring(0, string.indexOf(substring));
            subStrTrail = string.substring(subStrHead.length() + substring.length());
            string = subStrHead + subStrTrail;
        }
        return sum;
    }

    /**
     * 找最左边的左括号的镜像括号的位置，或者最右边的右括号的镜像括号的位置
     */
    private static int indexOfFirstMirrorBracket(String s, char c) {
        int indexresult = -1;
        int sum = 0;
        if (countsOfSubstringInString(s, "(") == countsOfSubstringInString(s, ")")) {
            if (c == '(') {
                int index = s.indexOf(c) - 1;
                do {
                    index++;
                    if (s.charAt(index) == '(')
                        sum++;
                    if (s.charAt(index) == ')')
                        sum--;
                } while (sum != 0 && index < s.length());
                indexresult = index - 1;
            }
            if (c == ')') {
                int index = s.lastIndexOf(c);
                do {
                    if (s.charAt(index) == ')') {
                        sum++;
                    }
                    if (s.charAt(index) == '(')
                        sum--;
                    index--;
                } while (sum != 0 && index >= 0);
                indexresult = index;
            }
        }
        return indexresult + 1;
    }

    /**
     * 指定位置的括号的镜像括号
     * 
     * @param s
     * @param c
     * @param index
     * @return
     */
    private static int indexOfOpecifiedMirrorBracket(String s, char c, int index) {
        int indexresult = -1;
        int sum = 0;
        int startIndex = -1;
        int endIndex = -1;
        StringBuffer sb = new StringBuffer(s);
        if (countsOfSubstringInString(s, "(") == countsOfSubstringInString(s, ")")) {
            if (index == 0 || index == s.length() - 1)
                return indexOfFirstMirrorBracket(s, c);
            else if (c == '(') {
                sum = countsOfSubstringInString(s.substring(0, index), String.valueOf(c));
                while (sum != 0) {
                    startIndex = sb.indexOf("(");
                    endIndex = indexOfFirstMirrorBracket(sb.toString(), c);
                    sb = sb.replace(startIndex, startIndex + 1, "a").replace(endIndex, endIndex + 1, "a");
                    sum--;
                }
                indexresult = indexOfFirstMirrorBracket(sb.toString(), c);
            } else if (c == ')') {
                sum = countsOfSubstringInString(s.substring(index + 1), String.valueOf(c));
                while (sum != 0) {
                    startIndex = sb.indexOf(")");
                    endIndex = indexOfFirstMirrorBracket(sb.toString(), c);
                    sb = sb.replace(startIndex, startIndex + 1, "a").replace(endIndex, endIndex + 1, "a");
                    sum--;
                }
                indexresult = indexOfFirstMirrorBracket(sb.toString(), c);
            }
        }
        return indexresult;
    }

    /**
     * 某个位置前面最近一个算术运算符的位置
     * 
     * @param s
     * @param specifiedIndex
     * @return
     */
    private static int indexOfLeftOperator(String s, int specifiedIndex) {
        int temp = -1;
        if (specifiedIndex >= 1 && s.substring(0, specifiedIndex).matches(".*(\\-|\\+|\\*|/).*")) {
            do {
                specifiedIndex--;
            } while (!String.valueOf(s.charAt(specifiedIndex)).matches("\\+|\\-|\\*|/"));
            temp = specifiedIndex;
        }
        return temp;
    }

    /**
     * 某个位置后面最近一个算术运算符的位置
     * 
     * @param s
     * @param specifiedIndex
     * @return
     */
    private static int indexOfRightOperator(String s, int specifiedIndex) {
        int temp = -1;
        boolean b = specifiedIndex >= 0 && specifiedIndex <= s.length() - 1
                && s.substring(specifiedIndex + 1).matches(".*(\\+|\\-|\\*|/).*");
        if (b) {
            while (!String.valueOf(s.charAt(specifiedIndex + 1)).matches("\\+|\\-|\\*|/")) {
                specifiedIndex++;
            }
            temp = specifiedIndex + 1;
        }
        return temp;
    }

 

    /**
     * 找数字结尾的字符串结束位置开始往前的一个完整数字的位置
     * 
     * @param result
     * @return
     */
    private static int indexOfNumberStart(String result) {
        int resultIndex = -1;
        int indexOfOperator = indexOfLeftOperator(result, result.length() - 1);// 该位置前面第一个运算符位置
        if (indexOfOperator == -1) {//前面没有运算符
            indexOfOperator = result.lastIndexOf('(');
            if (indexOfOperator == -1)
                resultIndex = 0;
            else
                resultIndex = indexOfOperator + 1;
        } else {
            if(result.charAt(indexOfOperator) == '-' && result.charAt(indexOfOperator + 1) != ' ')
                resultIndex = indexOfOperator;
            else{
                while (result.charAt(indexOfOperator + 1) == '(' || result.charAt(indexOfOperator + 1) == ' ')
                    indexOfOperator++;
                resultIndex = indexOfOperator + 1;
            }
        }
        return resultIndex;
    }

    /**
     * 找运算符位置往后完整数字的位置
     * 
     * @param result
     * @param index
     * @return
     */
    private static int indexOfNumberEnd(String result, int index) {
        int resultIndex = -1;
        int indexOfOperator = indexOfRightOperator(result, index + 1);
        String subStrStart = result.substring(0, index + 1);
        String subStrEnd = result.substring(index + 1);
        if (indexOfOperator == -1) {// 没有运算符
            if (result.substring(index + 1).contains("(")) {
                int startIndex = subStrStart.length() + subStrEnd.indexOf('(');
                int endIndex = indexOfOpecifiedMirrorBracket(result, '(', startIndex);
                resultIndex = endIndex + 1;
            } else if (indexOfOperator == -1) {
                while (index < result.length() && String.valueOf(result.charAt(index + 1)).matches("\\s|\\d|\\."))
                    index++;
                resultIndex = index + 1;
            }
        } else {
            if (result.substring(index + 1, indexOfOperator).contains("(")) {
                int startIndex = subStrStart.length() + subStrEnd.indexOf('(');
                int endIndex = indexOfOpecifiedMirrorBracket(result, '(', startIndex);
                resultIndex = endIndex + 1;
            } else
                resultIndex = indexOfOperator;
        }
        return resultIndex;
    }

    /**
     * 以数字结尾的字符串的最后一个完整的数字字符串
     * 
     * @param result
     * @return
     */
    private static String lastNumberOfStr(String result) {
        int indexTemp = indexOfLeftOperator(result, result.length() - 1);
        boolean b = String.valueOf(result.charAt(result.length() - 1)).matches("\\d");
        if (indexTemp <= result.length() - 1 && b) {
            while (!String.valueOf(result.charAt(indexTemp + 1)).matches("\\d"))
                indexTemp++;
            return result.substring(indexTemp + 1);
        } else
            return "";
    }

    /**
     * 右括号后面跟平方、求倒、开方
     * 
     * @param result
     * @param content
     * @return
     */
    private static String parseBrackets(String result, String content) {
        String temp = "";
        int startIndex = indexOfFirstMirrorBracket(result, ')');
        int indexOfOperator = indexOfLeftOperator(result, startIndex);
        String substrhead = "";
        String substrtrail = "";
        if (indexOfOperator == -1)
            substrtrail = result;
        else {
            substrhead = result.substring(0, indexOfOperator + 2);
            substrtrail = result.substring(indexOfOperator + 2);
        }
        if (content.equals("√")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "sqt" + substrtrail;
            else
                temp = substrhead + "sqt(" + substrtrail + ")";
        }
        if (content.equals("x²")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "sqr" + substrtrail;
            else
                temp = substrhead + "sqr(" + substrtrail + ")";
        }
        if (content.equals("x³")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "cube" + substrtrail;
            else
                temp = substrhead + "cube(" + substrtrail + ")";
        }
        if (content.equals("1/x")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "rec" + substrtrail;
            else
                temp = substrhead + "rec(" + substrtrail + ")";
        }
        if (content.equals("Abs")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "Abs" + substrtrail;
            else
                temp = substrhead + "Abs(" + substrtrail + ")";
        }
        if (content.equals("Int")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "Int" + substrtrail;
            else
                temp = substrhead + "Int(" + substrtrail + ")";
        }
        if (content.equals("sin")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "sin" + substrtrail;
            else
                temp = substrhead + "sin(" + substrtrail + ")";
        }
        if (content.equals("cos")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "cos" + substrtrail;
            else
                temp = substrhead + "cos(" + substrtrail + ")";
        }
        if (content.equals("log")) {
            if (substrtrail.startsWith("(") && substrtrail.endsWith(")")
                    && indexOfFirstMirrorBracket(substrtrail, '(') == substrtrail.length() - 1)
                temp = substrhead + "log" + substrtrail;
            else
                temp = substrhead + "log(" + substrtrail + ")";
        }
        if (temp.startsWith("(") && temp.endsWith(")") && indexOfFirstMirrorBracket(temp, '(') == temp.length() - 1)
            temp = temp.substring(1, temp.length() - 1);

        return temp;
    }

    /**
     * 数字结尾后面跟平方、求倒、开方
     * 
     * @param result
     * @param content
     * @return
     */
    private static String parseDigitEndWithFuncthion(String result, String content) {
        String temp = "";
        String contentTemp = "";
        if (content.equals("1/x"))
            contentTemp = "rec";
        if (content.equals("x³"))
            contentTemp = "cube";
        
        if (content.equals("x²"))
            contentTemp = "sqr";
        if (content.equals("x³"))
            contentTemp = "cube";
        if (content.equals("√"))
            contentTemp = "sqt";
        if (content.equals("log"))
            contentTemp = "log";
        if (content.equals("Abs"))
            contentTemp = "Abs";
        if (content.equals("Int"))
            contentTemp = "Int";
        if (content.equals("sin"))
            contentTemp = "sin";        
        if (content.equals("cos"))
            contentTemp = "cos"; 
        
        int startIndex = indexOfNumberStart(result);// 数字的开头
        String substrhead = result.substring(0, startIndex);
        String substrtrail = result.substring(startIndex);
        if (result.startsWith("-") && startIndex == 1) {
            if (contentTemp == "!")
                temp = "-" + result.substring(startIndex) + "!";
            else
                temp = contentTemp + "(" + result + ")";
        } else {
            if (contentTemp == "!") {
                if (substrtrail.matches(".*\\.\\d*[1-9]+$"))
                    temp = result;
                else
                    temp = substrhead + substrtrail + contentTemp;
            } else
                temp = substrhead + contentTemp + "(" + substrtrail + ")";
        }
        return temp;
    }

}
