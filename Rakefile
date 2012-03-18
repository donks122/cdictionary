
require 'rubygems'
require 'rake'
require 'nokogiri'

task :default => 'build:html'


class LCHelper

  CONTENT_FILE = 'commands.xml'

  class Node

    attr_accessor :children, :item

    def initialize( item ) 
      @item = item
      @children = [ ] 
    end

    def add_child( child )
      @children << child
    end
  end

  class << self

    def doc 
      @doc = Nokogiri::Slop( 
        File.open( CONTENT_FILE  ).read
      ) unless @doc

      @doc
    end

    def category_nav_for node_list 
      if node_list.size > 0
        markup = '<ul>'
        node_list.each do |i|

          next if i.item[ 'name'].nil?

          markup << "<li> <a href='\##{i.item[ 'name' ]}'>" << i.item[ 'display' ] << "</a>"
          markup << category_nav_for( i.children ) if i.children.size > 0
          markup << '</li>'
        end
        markup << '</ul>'
      end
    end

    def options_markup el

      return '' unless el.css( 'options > option' ).size > 0

      markup = '<section>'        <<
        '<span> Options </span>'  <<
        '<ul>'


      el.css( 'options > option' ).each do |j|
        markup << '<li>' << '<code>' << j[ 'name' ] << '</code>' << '<p>' << j.content << '</p>'
      end

      markup << '</ul> </section>'
      
      markup
    end

    def description_markup elem
      '<section>' << elem.css( 'description' ).first.content << '</section>'
    end

    def usage_markup elem
      return '' if ( item = elem.css( 'usage' ).first ).nil?

      '<code>' << item.content << '</code>'
    end

    def _invalid? elem
      return true if usage_markup( elem ).empty?

      false
    end


    def content_for node_list
      if node_list.size > 0 
        markup = '<section>' 
        node_list.each do |i|

          markup << '<ul>'
          i.item.css( 'command' ).each do |j|

            next if _invalid? j

            markup << '<li>' 
            markup << usage_markup( j )
            markup << options_markup( j )
            markup << description_markup( j )
            markup << '</li>'

          end
          markup << '</ul>'
        end
        markup << '</section>'
      end
    end

    def category_markup
      '<header>'                                << 
        '<section>'                             <<
          '<nav>'                               <<
            category_nav_for( @node.children )  <<
          '</nav>'                              <<
        '</section>'                            <<
      '</header>'
    end

    def content_markup
      '<section>'                               <<
        content_for( @node.children )           <<
      '</section>'
    end

    def parse_rest node_list
      if node_list.length > 0 
        node_list.each do |cat|
          item = Node.new( cat )
          @current_node.add_child( item ) 
          if cat.css( 'category' ).length > 0
            tmp = @current_node
            @current_node = item 
            parse_rest cat.css( 'category' )
            @current_node = tmp
          end
        end
      end
    end

    def parse 
      @current_node = @node = Node.new( self.doc.css( 'root' ).first )
      parse_rest( self.doc.css( 'root > category' ) )
      @node
    end

    def _r fname, dir = 'html'
      f = File.join( 'assets', ( dir || 'html' ), fname ) 
      File.open( f ).read  if File.exists? f
    end

    def get_header
      _r( 'head.html' )         << 
      _r( 'style.css', 'css' )  << 
      _r( 'app.js', 'js' )      <<
      _r( 'top.html' )
    end

    def get_footer 
      _r( 'foot.html' )
    end

  end
end



namespace 'build' do

  task :html do
    LCHelper.parse
    File.open( 'build.html', 'w' ) do |f|
      f.write LCHelper.get_header
      f.write LCHelper.category_markup
      f.write LCHelper.content_markup
      f.write LCHelper.get_footer
    end
  end

end



