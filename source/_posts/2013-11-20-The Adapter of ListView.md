title: The Adapter of ListView: Just adapt data to view, don’t do anything else
date: 2013-11-20
tags: Android
---
The design of SimpleAdapter is not good in my opinion.

An adapter should just adapter the data to view, not care to inflate xml to create Layout View, hold the member Views of the layout view, and, fill the datas to each member View one by one. SimpleAdapter does these things all together. It is high coupling design.

The solution is a ItemViewHolder class, which inflate and hold the layout view and its member views, and parse datas to fill them  to member views one by one. And the most cool is, it’s layout view of ItemViewHolder set “this” as its tag. So that you can get the ItemViewHolder instance of the convertView in getView(…, View convertView, …) function.

See below code snippets.

In Adapter, passed in a List as data list.

{% code %}
List<Data> dataList;  
{% endcode %}

and override getView() like this:

<!-- more -->

{% code %}
@Override  
public View getView(int position, View convertView, ViewGroup parent) {  
    ItemViewHolder item = null;  
  
    if(convertView == null) {  
        item = new ItemViewHolder(context);  
        convertView = item.getLayoutView();  
    } else {  
        item = (ItemViewHolder)convertView.getTag();  
    }  
  
    item.setItemData(dataList.get(position));  
  
    return convertView;  
}  
{% endcode %}

ItemViewHolder is the views holder of the convertView.

{% code %}
public class ItemViewHolder {  
  
    //The layout View of the item of the ListView.  
    private View layoutView = null;  
    //The member Views to display data.  
    private TextView textView = null;  
    private ImageView imageView = null;  
    … …  
  
    public ItemViewHolder (Context context) {  
        super(context);  
  
         initUI();  
    }  
  
    public View getLayoutView() {  
        return layoutView;  
    }  
  
    public void setItemData(Data data) {  
        textView.setText(data.getText());  
        imageView.setImage(data.getImage());  
        … …  
    }  
  
    private void initUI() {  
         LayoutInflater inflater = LayoutInflater.from(mContext);  
         layoutView = inflater.inflate(R.layout.item_view_layout, null);  
        //  
        textView = (TextView) layoutView.findViewById(R.id.textview);  
        imageView = (ImageView)layoutView.findViewById(R.id.imageview);  
       //This is the most important code.  
       layoutView.setTag(this);  
    }  
}  
{% endcode %}
